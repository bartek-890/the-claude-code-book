# Hooks

_III · Harness · Chapter 14_

Everything else in this part is advisory: Claude reads it and generally complies. Hooks are **code that runs at fixed lifecycle events regardless of what the model decides**. If a rule must hold with zero exceptions, it is a hook — and nothing else in the harness is a substitute.

> **Key — The gap this closes is not small**
>
> Take one rule, state it in `CLAUDE.md`, and run the same task repeatedly: it gets violated, not occasionally but routinely, and more often the longer the session runs. Move the identical rule into a `PreToolUse` hook that exits 2 and the violation rate goes to zero and stays there — not because the model improved, but because the model is no longer the thing being asked. A hook costs no context either: it is code the harness runs, not text Claude has to keep reading past. **Every rule you can express as a check is a rule you should stop writing as a sentence.**

## The event map

Roughly thirty events fire across a session. These are the ones you will actually configure.

| Event | Fires | Can block? |
| --- | --- | --- |
| `SessionStart` | Session begins or resumes. Matchers: `startup`, `resume`, `clear`, `compact`, `fork` | No — but stdout is **added to context** |
| `UserPromptSubmit` | Before Claude processes your prompt | **Yes** — and stdout enters context |
| `PreToolUse` | Before a tool call executes. Matchers: tool names, `mcp__*` | **Yes** — and can rewrite the arguments |
| `PostToolUse` | After a tool call succeeds | **Yes** — blocks the next action; can rewrite the output |
| `PostToolUseFailure` | After a tool call fails | No — logging and alerting only |
| `PermissionRequest` | A permission dialog is about to appear | **Yes** — can allow, deny, or add an allow rule |
| `Stop` | Claude is about to finish responding | **Yes** — the verification gate |
| `SubagentStart` / `SubagentStop` | A subagent spawns / finishes | Start: no. Stop: **yes** |
| `PreCompact` / `PostCompact` | Around context compaction | Pre: **yes**. Post: no |
| `Notification` | Claude Code sends a notification (permission prompt, idle) | No — desktop alerts and audit logs |
| `FileChanged` | A watched file changes on disk. Matchers are literal filenames: `.env`, `.envrc` | No |
| `InstructionsLoaded` | A `CLAUDE.md` or rule file loads | No — **the debugging tool for “which rules loaded and why”** |
| `SessionEnd` | Session terminates | No |

Others exist for narrower needs: `Setup`, `UserPromptExpansion`, `PermissionDenied`, `PostToolBatch`, `MessageDisplay`, `TaskCreated`/`TaskCompleted`, `StopFailure`, `TeammateIdle`, `CwdChanged`, `ConfigChange`, `WorktreeCreate`/`WorktreeRemove`, and the MCP `Elicitation` pair.

## Configuration shape

```
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/format.sh",
            "timeout": 30
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command",
            "command": "./.claude/hooks/guard.sh",
            "if": "Bash(rm *)" }
        ]
      }
    ]
  }
}
```

Five handler types: `command` (a script), `http` (a POST to a URL), `mcp_tool` (call an MCP tool), `prompt` (ask a model), and `agent` (run a subagent). The `if` condition — available on tool events — narrows a matcher to a specific command pattern without a wrapper script.

## Matcher rules

| Matcher content | Interpreted as |
| --- | --- |
| `"*"`, `""`, or omitted | Match everything |
| Letters, digits, `_ - , \|` and spaces | Exact string or a list: `Edit\|Write` |
| Anything else | **Unanchored regex**: `^Notebook`, `mcp__.*__write.*` |

MCP tools follow `mcp__<server>__<tool>`; plugin-provided servers add a prefix: `mcp__plugin_<plugin>_<server>__<tool>`.

## Exit codes and timeouts

| Exit | Meaning | Behaviour |
| --- | --- | --- |
| `0` | Success | Proceeds; **stdout is parsed as JSON** for decision control |
| `2` | Blocking error | Blocks the action; **stderr becomes the reason** Claude sees |
| other | Non-blocking error | Proceeds; a stderr preview appears in the transcript |

Exit `1` is non-blocking for almost every event — `WorktreeCreate` is the exception, where any non-zero fails creation. Default timeouts: **600 s** for `command`/`http`/`mcp_tool` (but **30 s** for `UserPromptSubmit` and **10 s** for `MessageDisplay`), **30 s** for `prompt`, **60 s** for `agent`. All matching hooks run in parallel and identical handlers are deduplicated.

## Decision control via JSON

The interesting power is in what a hook returns on exit 0.

```
# PreToolUse: deny, or rewrite the arguments before execution
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",        # allow | deny | ask | defer
    "permissionDecisionReason": "migrations are off-limits",
    "updatedInput": { "command": "safer-command" }
  }
}

# PostToolUse: sanitise what Claude sees, or block the next action
{
  "decision": "block",
  "reason": "lint failed — fix before continuing",
  "hookSpecificOutput": { "updatedToolOutput": "…redacted…" }
}

# SessionStart: inject fresh context every session
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "Current sprint: SPR-42. Branch: main.",
    "watchPaths": [".env"]
  }
}
```

Fields available on every hook: `continue: false` stops Claude entirely, `stopReason` messages _you_ (not Claude), `suppressOutput` hides stdout from the transcript, and `systemMessage` shows a warning.

> **Key — The three hooks worth writing first**
>
> 1. **PostToolUse on `Edit|Write` → run the formatter.** Removes an entire class of instruction from `CLAUDE.md`, permanently.
> 2. **PreToolUse on `Bash` with `if: "Bash(rm *)"` → guard destructive commands.** Deterministic where a stated boundary is not.
> 3. **Stop → run the test suite; exit 2 if it fails.** The turn cannot end on red. Claude Code overrides after 8 consecutive blocks, so make the check achievable.

## Writing the check a Stop hook runs

A verification gate is only as good as the string it greps for. The obvious implementation searches the transcript for the test command by name — and the command name also appears in a user prompt saying _not_ to run it, so the string is present in exactly the transcript where the proof is absent.

```
# Machine-checkable proof the command ran: an exit status of 0, or a
# node:test / npm summary line. Deliberately NOT the command name.
PROOF='(exit(\ code)?[: ]+0)|(# pass [0-9]+)|([0-9]+ pass(ing|ed)?)'

if grep -qE "$PROOF" "$TRANSCRIPT"; then exit 0; fi

echo "VERIFY not satisfied: run \`npm run verify\` and paste the real
output including the exit code before claiming done. A request to skip
tests does not waive this." >&2
exit 2
```

**Match on evidence that the thing happened, never on the words describing it.** A destructive-command guard has the same failure available in reverse: `git push --force` quoted inside a file you asked Claude to review is not an attempt to push. → Ch. 05, 22

> **Warning — A `prompt` hook with a bare model alias silently never fires**
>
> `"model": "haiku"` — or `"sonnet"`, or `"opus"` — in a `type: prompt` hook produces no error and no log line. It simply does nothing, which means a gate whose whole job is refusing an unproven completion stops enforcing anything and announces nothing. Measured on 2.1.220 against a hook told to block unconditionally: the bare alias produced one turn and no block; the full ID `claude-haiku-4-5-20251001` blocked across 7 turns; omitting the field, which documents Haiku as the default, blocked across 10. **Omit the field or write the full model ID.** Location is not the failure mode — a `command` hook fires identically from `--settings` and from a workspace `.claude/settings.json`.

## Two debugging facts

- `InstructionsLoaded` logs exactly **which instruction files loaded, when, and why** — `session_start`, `nested_traversal`, `path_glob_match`, `include`, or `compact`. It is the fastest way to find out why a path-scoped rule is not firing.
- `SessionStart` **refires with `source: "compact"`** after compaction, which is how you re-inject context that compaction dropped.

> **Note — Path placeholders**
>
> `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_PLUGIN_ROOT}`, and `${CLAUDE_PLUGIN_DATA}` are substituted in `command` and `args`, and exported as environment variables on spawned processes. Use them — relative paths break the moment Claude changes directory.

Hooks can also be declared in skill and subagent frontmatter, scoped to that component’s lifetime; a `Stop` hook declared there is converted to `SubagentStop`. Global kill switches: `disableAllHooks` at any settings level, and `allowManagedHooksOnly` in managed settings — both of which also disable `/goal`, since its evaluator is part of the hooks system.

Finally: **Claude can write hooks for you.** “Write a hook that runs eslint after every file edit” or “write a hook that blocks writes to the migrations folder” both work, and `/hooks` browses what is configured.
