# The CLI

_II · Controls · Chapter 09_

Everything before this chapter happens inside a session. This is how sessions start, how they run without you, and how Claude Code becomes a component in a script rather than an application you sit in front of.

## Starting a session

| Command | Effect |
| --- | --- |
| `claude` | Interactive session |
| `claude "query"` | Interactive, with an opening prompt |
| `claude -p "query"` | **Print mode** — non-interactive, prints, exits |
| `cat file \| claude -p "…"` | Process piped content |
| `claude -c` | Continue the most recent conversation in this directory |
| `claude -r "<session>" "query"` | Resume a specific session by ID or name |
| `claude --resume` | Interactive session picker |

## The flags that change what happens

| Flag | Why you would use it |
| --- | --- |
| `--permission-mode <mode>` | Start in `plan`, `acceptEdits`, `auto`, `dontAsk`, or `bypassPermissions`. → Ch. 05 |
| `--model <alias\|id>` | `sonnet` · `opus` · `haiku` · `fable` or a full ID |
| `--effort <level>` | One session at a chosen effort level. → Ch. 28 |
| `--fallback-model` | Automatic fallback when the primary is overloaded |
| `--add-dir <path>` | Grant file access beyond the working directory |
| `--allowedTools` / `--disallowedTools` | Per-run permission rules. **Essential for unattended loops.** |
| `--tools` | Restrict which built-in tools exist at all this session |
| `--output-format` | `text` · `json` · `stream-json` |
| `--json-schema` | Validated JSON output matching a schema you supply |
| `--max-turns <n>` | Hard cap on agentic turns |
| `--max-budget-usd <n>` | **Stop after spending this much.** The cheapest insurance in the CLI — but **print mode only**, so it bounds `claude -p` runs and does nothing in an interactive session. Subagent spend counts against the cap. |
| `--append-system-prompt` | Append text to the system prompt — survives compaction, unlike chat instructions |
| `--system-prompt` / `--system-prompt-file` | Replace the system prompt entirely |
| `--agents <json>` | Define subagents inline without files. → Ch. 13 |
| `--agent <name>` | Run the whole session _as_ a named agent |
| `--mcp-config` / `--strict-mcp-config` | Load MCP servers from a file; ignore all others |
| `--settings <path\|json>` | Point at a settings file or inline JSON |
| `--setting-sources user,project` | Load only the scopes you name |
| `--session-id <uuid>` | Pin a session ID — makes automation resumable by construction |
| `--fork-session` | Resume into a new session ID rather than reusing the original |
| `--worktree [name]` / `-w` | Run in a git worktree, named if you name it. Add `--tmux` to launch it in its own tmux session. → Ch. 23 |
| `--name <name>` | Name the session up front, so `--resume` lists workstreams instead of timestamps |
| `--bg` / `--background` | Start as a background agent and return immediately |
| `--verbose` | Full turn-by-turn output. On while developing a script, off in production. |
| `--debug` | Capture a debug log for the whole session. `/debug` mid-session only logs from that point on, so pass this when you already know you will need it. |
| `--safe-mode` | All customisations disabled — the configuration-debugging baseline |
| `--bare` | Skip auto-discovery for the fastest start |

## Subcommands worth knowing

| Command | Effect |
| --- | --- |
| `claude doctor` | Installation and settings diagnostics |
| `claude mcp …` | Configure MCP servers. → Ch. 15 |
| `claude plugin` | Manage plugins |
| `claude setup-token` | Long-lived OAuth token for CI and scripts |
| `claude agents` | Agent view — monitor parallel background sessions |
| `claude attach <id>` / `logs` / `stop` / `respawn` | Drive background sessions from the shell |
| `claude project purge [path]` | Delete local Claude Code state for a project |
| `claude update` | Update to the latest version |

## Print mode: three output shapes

```
# plain text — for humans and simple pipes
claude -p "Explain what this project does"

# one JSON object with a `result` field — for scripts
claude -p "List all API endpoints" --output-format json

# one JSON object per line, starting with an init event — for streaming
claude -p "Analyze this log file" --output-format stream-json --verbose
```

A `-p` run still creates a resumable session unless you pass `--no-session-persistence`. Interactive-only affordances — multiple-choice questions, plan approval — are disabled in `-p`, so a run cannot hang waiting for a human.

> **Key — The unattended-run starting point**
>
> ```
> claude -p "/goal every file in src/legacy compiles and \`npm test\` exits 0" \
>   --permission-mode auto \
>   --allowedTools "Edit,Read,Grep,Glob,Bash(npm test *),Bash(npx tsc *)" \
>   --max-turns 40 \
>   --max-budget-usd 8 \
>   --output-format stream-json --verbose
> ```
>
> Five bounds on one command: a machine-checked stop condition, a classifier on every action, an explicit tool allowlist, a turn cap, and a spend cap. Remove any one and you have an unbounded process holding your credentials.
