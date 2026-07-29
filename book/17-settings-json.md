# settings.json

_III · Harness · Chapter 17_

One file format, five scopes, and a precedence order that explains most “why isn’t my setting taking effect?” confusion. Everything configurable in Claude Code eventually lands here.

## Scopes, highest precedence first

| Scope | Location | Shared |
| --- | --- | --- |
| **Managed** | MDM / plist / registry / `managed-settings.json` | IT-deployed. **Cannot be overridden.** |
| **CLI arguments** | `--settings`, individual flags | This invocation only |
| **Local** | `.claude/settings.local.json` | No — gitignored |
| **Project** | `.claude/settings.json` | Yes — committed |
| **User** | `~/.claude/settings.json` | No — all your projects |

Permission rules **merge** across scopes rather than overriding; the precedence order applies to scalar values. A lower scope can never loosen a managed deny. Run `/status` to see which sources actually loaded.

## The keys worth knowing

| Key | Effect |
| --- | --- |
| `model` · `fallbackModel` | Active model; chain of fallbacks when the primary is overloaded |
| `effortLevel` | `low` · `medium` · `high` · `xhigh` (not `max`) |
| `fastMode` | Opus-tier latency mode. → Ch. 28 |
| `permissions.allow` / `.deny` / `.ask` | The `ToolName(specifier)` rules. → Ch. 04 |
| `sandbox.*` | OS isolation, credentials, network allowlists. → Ch. 05 |
| `hooks` | Lifecycle hooks. → Ch. 14 |
| `env` | Environment variables applied to every session |
| `autoMemoryEnabled` · `autoMemoryDirectory` | Auto-memory control. → Ch. 11 |
| `claudeMd` (managed only) | Org-wide instructions inline, without deploying a separate file |
| `claudeMdExcludes` | Glob patterns of memory files to skip. Arrays merge across layers. |
| `cleanupPeriodDays` | Session, checkpoint, and subagent-transcript retention. Default 30. |
| `includeCoAuthoredBy` · `attribution` | Commit and PR message customisation |
| `enableAllProjectMcpServers` | Auto-approve project-scoped MCP servers |
| `disabledMcpjsonServers` · `enabledMcpjsonServers` | Block or allowlist specific servers |
| `statusLine` | Custom status line — the place to surface live context usage |
| `outputStyle` | Output formatting mode. Read at startup; needs a restart. |
| `editorMode` | `normal` or `vim` |
| `disableAllHooks` | Kill switch for hooks, custom status line — and `/goal` |
| `disableBundledSkills` | Turn off the built-in skills |
| `autoUpdatesChannel` | `latest` or `stable` |

## Environment variables

| Variable | Effect |
| --- | --- |
| `CLAUDE_CODE_EFFORT_LEVEL` | Effort level — **highest precedence, overrides everything** |
| `MAX_THINKING_TOKENS` | Cap reasoning tokens; `0` disables thinking on the API |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | Revert fixed-budget models to the older thinking mode |
| `DISABLE_AUTO_COMPACT` | Turn off automatic compaction |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | Turn off auto memory |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | Turn off rewind checkpoints |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | Load memory files from `--add-dir` directories |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | Scrub provider credentials from **every** subprocess environment |
| `CLAUDE_CODE_NEW_INIT` | Interactive multi-phase `/init` |
| `DISABLE_AUTOUPDATER` · `CLAUDE_CODE_ENABLE_TELEMETRY` | Updates and telemetry |

## A working project settings file

```
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": ["Bash(pnpm test *)", "Bash(pnpm lint)", "Bash(git status)"],
    "ask":   ["Bash(git push *)"],
    "deny":  ["Read(./.env*)", "Read(~/.ssh/**)", "Read(~/.aws/**)",
              "Edit(migrations/**)"]
  },
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{ "type": "command",
                   "command": "pnpm prettier --write \"$CLAUDE_FILE\"" }]
    }]
  },
  "cleanupPeriodDays": 30
}
```

## The status line: put context usage where you can see it

Chapter 3 argues that the context window is the resource that decides output quality. The status line is how you stop having to run `/context` to find out where you are. It runs any shell script, receives JSON session data on stdin, and prints whatever you want.

```
{
  "statusLine": { "type": "command", "command": "~/.claude/statusline.sh", "padding": 2 }
}
```

The one-liner version, if you do not want a script:

```
{
  "statusLine": {
    "type": "command",
    "command": "jq -r '\"[\\(.model.display_name)] \\(.context_window.used_percentage // 0)% ctx\"'"
  }
}
```

The fields worth reaching for:

| Field | What it holds |
| --- | --- |
| `context_window.used_percentage` | Pre-calculated percentage used — the one number that matters |
| `context_window.context_window_size` | 200,000 by default, 1,000,000 on extended-context models |
| `context_window.total_input_tokens` | Tokens currently in the window, from the last API response. Includes cache reads and writes. v2.1.132+ — before that these were cumulative session totals. |
| `cost.total_cost_usd` | Estimated session cost, computed client-side. v2.1.211+ resets to $0 on `/clear`. |
| `model.display_name` | Which model is actually running — useful after a fallback |
| `exceeds_200k_tokens` | Fixed 200K threshold flag, regardless of the real window size |

Two `null` cases to guard: `current_usage` is null before the first API call and again after `/compact` until the next one, and the percentage fields can be null early in a session. Hence `// 0` in the example. `subagentStatusLine` configures the same thing for subagents; `/statusline` asks Claude to write or remove one for you.

> **Key — The three settings that change behaviour with zero discipline**
>
> 1. **Credential deny rules.** There is no built-in list; by default the agent can read `~/.ssh` and `~/.aws/credentials`. These close it from the first session.
> 2. **An `Edit(...)` deny on migrations, auth, or billing.** Off-limits zones are a security control, not a hygiene habit.
> 3. **A format-on-edit hook.** Deletes an entire category of instruction from `CLAUDE.md`, permanently.

> **Note — When a setting seems to do nothing**
>
> Run `/status` for active setting sources, then `/doctor`, then start with `--safe-mode` to confirm your configuration is the cause. Most settings reload live; `model` and `outputStyle` require a restart.
