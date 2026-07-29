# The startup tax

_I · Mechanism · Chapter 02_

Before your first keystroke, a measurable amount of your context window is already spent. You pay it on every new session and every `/clear`. It is the one cost that competes with your actual task on _every single turn_, and it is almost entirely under your control.

## What loads at session start

| Surface | Size behaviour | Notes |
| --- | --- | --- |
| System prompt | Fixed | Core behaviour, tool-use instructions, response formatting. Spends roughly **50 directives** before your file is read. |
| Project-root `CLAUDE.md` | Full file, no cap | Loaded in full regardless of length. Re-injected from disk after compaction. |
| Parent-directory `CLAUDE.md` | Full file, each level | Every ancestor directory up to the filesystem root, root-down order. |
| `CLAUDE.local.md` | Full file | Appended after `CLAUDE.md` at each level. Gitignored by convention. |
| `.claude/rules/*.md` without `paths:` | Full file | Same priority as `.claude/CLAUDE.md`. Discovered recursively. |
| `@path` imports | Full file, ≤4 hops deep | Expanded at launch. Splitting into imports organises; it does **not** reduce context. |
| Auto memory `MEMORY.md` | First **200 lines or 25 KB**, whichever is smaller | Hard-truncated. Topic files in the same directory are **not** loaded until read. |
| MCP tool _names_ | Names only by default | Full schemas deferred to the `ToolSearch` tool. Only **connected** servers cost anything. |
| Skill _descriptions_ | One line each, ≤1,536 chars | Bodies load only on invocation. |
| Subagent names + descriptions | One line each | Every registered agent costs a line, used or not. |
| Output style / `--append-system-prompt` | Full text | Both enter the system prompt itself. |
| Git status snapshot | Small | Taken at session start; suppressible via `includeGitInstructions: false`. |

> **Key — Measure yours, don’t guess**
>
> `/context` shows the live breakdown by category with optimisation suggestions. `/memory` shows which memory files actually loaded. `/mcp` shows per-server cost. Run all three once on your main project before you change anything — most people are surprised by which line dominates.

## The instruction budget

The tax is not only tokens. Practitioners peg reliable instruction-following for frontier thinking models at roughly **150–200 discrete instructions** — a rule of thumb, not a published measurement; see Appendix F. Claude Code’s own system prompt spends about **50** of them before your file is read. Every line you add to `CLAUDE.md` dilutes the others — which is why a bloated rules file does not produce a more obedient agent, it produces a less obedient one. Chapter 10 covers the fix.

## Four cuts, in order of payoff

1. **Disconnect MCP servers you are not using this week**

   Every connected server costs tool names at startup, and a chatty server costs far more. Deferred schema loading already helps enormously — Anthropic measured the same pattern on the API side cutting upfront tool tokens from ~77K to ~8.7K, an **85% reduction** (Appendix F, which also marks how far that number travels) — but names are not free either. Disconnect rather than deny.

2. **Hide manual-only skills from the startup index**

   Set `disable-model-invocation: true` in a skill’s frontmatter and its description stays out of the always-loaded index. You can still run it with `/skill-name`. Use it for every skill you invoke deliberately rather than one you want Claude to reach for on its own.

3. **Move path-specific rules behind `paths:` frontmatter**

   A rule in `.claude/rules/` with a `paths:` glob loads only when Claude reads a matching file. Rules without it load unconditionally, every session, for every task.

4. **Cut `CLAUDE.md` to what cannot be inferred**

   Target under 200 lines; the lean shape is closer to 60. The one-minute test: if a competent new contributor could work it out from the repo in a minute, it does not belong in the file. → Ch. 10

> **Warning — The monorepo trap**
>
> Claude Code walks _up_ the directory tree loading every `CLAUDE.md` it finds. In a large monorepo you can inherit three other teams’ rule files without noticing. Fix it with `claudeMdExcludes` in `.claude/settings.local.json`, which takes glob patterns matched against absolute paths and merges across settings layers.
>
> ```
> {
>   "claudeMdExcludes": [
>     "**/monorepo/CLAUDE.md",
>     "/home/user/monorepo/other-team/.claude/rules/**"
>   ]
> }
> ```
>
> Managed-policy `CLAUDE.md` files cannot be excluded — that is deliberate.

## A faster start when you don’t need any of it

Two flags exist for the cases where the harness is the problem:

- **claude --bare** — Skip auto-discovery entirely; fastest possible start
- **claude --safe-mode** — All customisations disabled — the debugging baseline
- **claude --setting-sources user** — Load only the scopes you name (user / project / local)

`--safe-mode` is the first move when Claude starts behaving strangely and you suspect your own configuration. If the behaviour disappears, bisect your settings; if it persists, it is the model or the task.
