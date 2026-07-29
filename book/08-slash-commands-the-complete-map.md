# Slash commands: the complete map

_II · Controls · Chapter 08_

The official list is long and grows weekly. This is the whole surface, grouped by what you are trying to do — with the ones worth memorising called out. Commands marked SKILL are bundled skills rather than hard-coded commands, which means Claude can also invoke them on its own. Anything with a version tag is recent enough that an older install will answer `Unknown command`.

## Session lifecycle

| Command | Aliases | What it does |
| --- | --- | --- |
| `/init` | — | Generate a starter `CLAUDE.md` from your codebase. If one exists, suggests improvements instead of overwriting. `CLAUDE_CODE_NEW_INIT=1` switches to an interactive flow that also walks through skills, hooks, and personal memory. |
| `/clear [name]` | `/reset` `/new` | New conversation, empty context. Pass a name to label the outgoing one in the `/resume` picker. |
| `/resume [id]` | `/continue` | Return to an earlier conversation, by ID, by name, or from the picker. Background sessions appear marked `bg`; a running one has to be attached or stopped from `claude agents` first. |
| `/rename [name]` | — | Name the session and show it on the prompt bar. Without a name, generates one from the history. |
| `/recap` | — | One-line summary of the session so far, on demand. |
| `/branch [name]` | — | Branch the conversation here and switch into the branch. The original stays, reachable from `/resume`. |
| `/fork [prompt]` | — | Copy the conversation into a new background session while you keep working here. v2.1.212+ |
| `/subtask <task>` | — | Forked subagent: inherits the full conversation, works in the background, and returns its result _into this conversation_. v2.1.212+ |
| `/background [prompt]` | `/bg` | Detach this session to run as a background agent and free the terminal. |
| `/stop` | — | Stop the background session you are attached to. Transcript and worktree are kept. |
| `/exit` | `/quit` | Exit the CLI. Inside an attached background session this only detaches — the session keeps running. |

> **Note — Four commands that all sound like “copy this conversation”**
>
> | Command | Where the copy runs | Where you end up |
> | --- | --- | --- |
> | `/branch` | Foreground, same terminal | You move into the branch; the original waits |
> | `/fork` | Background, independent | You stay; the copy never reports back |
> | `/subtask` | Background subagent | You stay; **the result lands in this conversation** |
> | `/background` | This session, detached | Terminal freed; monitor with `claude agents` |
>
> Pick by the last column. `/subtask` is the one you want when the answer matters to what you are doing now; `/fork` is the one you want when it does not.

## Context and memory — the group that pays for itself

| Command | Aliases | What it does |
| --- | --- | --- |
| `/context [all]` | — | **Colored grid of what is filling the window**, with optimisation suggestions and a warning, with the exact number, when the conversation has run past the limit. Run this before you optimise anything. |
| `/compact [instr]` | — | Summarise the conversation to free space. Always pass instructions. |
| `/memory` | — | Browse and edit `CLAUDE.md` files and auto-memory; toggle auto memory on or off. |
| `/btw [question]` | — | Side question in a dismissible overlay that **never enters history**. Run it bare to reopen the last one. |
| `/copy [N]` | — | Copy the last assistant response; `/copy 2` takes the one before it. With code blocks present you get a picker — press `w` to write the selection to a file instead, which is how you get output off a remote box. |
| `/export [file]` | — | Export the conversation as plain text. |
| `/rewind` | `/checkpoint` `/undo` | Roll code and/or conversation back to a checkpoint, or summarise from a point. |
| `/focus` | — | Collapse the view to your prompt, a one-line tool summary with diffstats, and the answer. Persists across sessions; fullscreen rendering only. |

## Model and effort

| Command | Aliases | What it does |
| --- | --- | --- |
| `/model [name]` | — | Switch model and save as default; press `s` on a row to switch for this session only. Confirms first when the conversation already has output, because the next reply re-reads the whole history uncached. |
| `/effort [level]` | — | Set effort: `low` `medium` `high` `xhigh` `max` `ultracode`. `/effort auto` resets to the model default. Applies immediately, without waiting for the current response. → Ch. 28 |
| `/fast [on\|off]` | — | Toggle fast mode — same model, more output speed, higher price. → Ch. 28 |
| `/advisor [model]` | — | Consult a second model at decision points during a task. Takes `opus`, `sonnet`, or a full model ID; `off` disables it. |

## Harness configuration

| Command | Aliases | What it does |
| --- | --- | --- |
| `/config [k=v]` | `/settings` | Open settings, or set one directly: `/config theme=dark`, `/config thinking=false`. `/config --help` lists every settable key. |
| `/permissions` | `/allowed-tools` | Manage allow / ask / deny rules, working directories, and recent auto-mode denials. |
| `/sandbox` | — | Toggle OS-level sandboxing. Supported platforms only. → Ch. 05 |
| `/add-dir <path>` | — | Grant file access to another directory for this session. Most `.claude/` configuration in that directory is **not** picked up. |
| `/cd <path>` | — | Move the session to a new working directory. **The prompt cache survives** — the new `CLAUDE.md` is appended as a message rather than rebuilt into the system prompt. v2.1.169+ |
| `/hooks` | — | View configured hooks for tool events. → Ch. 14 |
| `/mcp` | — | Manage MCP servers and OAuth. Takes `reconnect <server>`, or `enable`/`disable` with a name or `all`, without opening the dialog. → Ch. 15 |
| `/skills` | — | List skills. **Press `t` to sort by token count** — the fastest way to find what is inflating your startup — and `Space` to cycle a skill’s visibility. |
| `/agents` | — | Subagent configurations. From v2.1.198 it points you at `.claude/agents/` and at asking Claude to write one. → Ch. 13 |
| `/plugin [sub]` | — | Plugin menu, or `list` / `install` / `enable` / `disable` directly. → Ch. 16 |
| `/reload-skills` | — | Re-scan skill directories so edits made on disk apply without restarting. |
| `/reload-plugins` | — | Same for plugins. Warns and skips when the reload would change MCP tools and **invalidate the prompt cache**; `--force` overrides. |
| `/statusline` | — | Configure the status line — describe what you want, or run it bare to derive it from your shell prompt. → Ch. 17 |
| `/tui [mode]` | — | `fullscreen` switches to the flicker-free alt-screen renderer and relaunches with the conversation intact. |

## Workflow and automation

| Command | Type | What it does |
| --- | --- | --- |
| `/plan [task]` | built-in | Enter plan mode, optionally with a task description. → Ch. 18 |
| `/goal <condition>` | built-in | Keep working until a fresh evaluator confirms the condition. → Ch. 22 |
| `/loop [interval] [prompt]` | SKILL | Run a prompt repeatedly while the session is open — `/loop 5m check if the deploy finished`. Omit the interval and Claude self-paces; omit the prompt and it runs `.claude/loop.md`. Alias `/proactive`. |
| `/schedule [desc]` | built-in | Create, list, or run routines on Anthropic-managed infrastructure — the version of `/loop` that survives closing the laptop. Alias `/routines`. |
| `/batch <instruction>` | SKILL | Decompose a large change into 5–30 units, one background subagent per unit, each in its own worktree with its own PR. → Ch. 24 |
| `/tasks` | built-in | Background work in this session: running shells and subagents, finished ones included. Also `/bashes`. |
| `/workflows` | built-in | Watch, pause, resume, or save running workflows. |
| `/run` · `/verify` | SKILL | Launch the app and drive it, or confirm a change works by observing the running app rather than a green test. `/run-skill-generator` teaches both how your project starts. v2.1.145+ |
| `/deep-research <q>` | workflow | Fan out web searches, cross-check sources, synthesise a cited report. |
| `/ultraplan <prompt>` | built-in | Draft a plan in a cloud session, review it in the browser, then execute remotely or send it back to the terminal. |
| `/claude-api` | SKILL | Load Claude API reference for your language. `/claude-api migrate` updates model IDs and changed parameters across your code. |

## Review and quality

| Command | Type | What it does |
| --- | --- | --- |
| `/diff` | built-in | Interactive viewer for uncommitted changes and per-turn diffs. Left/right switches between the git diff and individual turns; from v2.1.198 it also refreshes when git state changes outside the session. |
| `/code-review` | SKILL | Review the current diff in a **fresh subagent** for correctness bugs and cleanup. Takes an effort level, `--fix`, `--comment` to post inline PR comments, and a target. `/code-review ultra` runs the deep cloud review. |
| `/review [PR]` | built-in | Fast single-pass, read-only review of a GitHub PR. Text after the number becomes extra instructions. |
| `/simplify [target]` | SKILL | Four parallel agents on reuse, simplification, efficiency, and level of abstraction — then it applies the fixes. **It does not look for bugs**; that is `/code-review`. |
| `/security-review` | SKILL | Review the branch diff against origin’s default branch for injection, auth, and data-exposure risks. Needs an `origin` remote. |
| `/autofix-pr [prompt]` | built-in | Cloud session that watches the current branch’s PR and pushes fixes when CI fails or a reviewer comments. Needs the `gh` CLI. |

## Diagnostics and support

| Command | Aliases | What it does |
| --- | --- | --- |
| `/doctor` | `/checkup` | Setup checkup with auto-fixes: duplicate installs, PATH, unparseable settings, unused skills and MCP servers against their context cost, slow hooks — including a proposed trim of a bloated `CLAUDE.md` v2.1.206+. Reports first, changes nothing without confirmation. |
| `/debug` | — | Enable debug logging **from this point forward** and read the log back. Logging is off unless you started with `claude --debug`, so run it _before_ reproducing the problem. |
| `/status` | — | Version, model, account, connectivity. Works while Claude is responding. |
| `/usage` | `/cost` `/stats` | Session cost, plan limits, and — on paid plans — a breakdown by skill, subagent, plugin, and MCP server. → Ch. 29 |
| `/insights` | — | Report over your own session history: project areas, interaction patterns, friction points. The evidence for what to fix in your harness. |
| `/fewer-permission-prompts` | — | Scan transcripts for repeated read-only calls and write a prioritised allowlist into project settings. |
| `/team-onboarding` | — | Turn 30 days of your usage into a setup guide a teammate can paste as a first message. |
| `/release-notes` | — | Changelog in a version picker. From v2.1.208 the notes stay out of the conversation Claude sees. |
| `/help` | — | Help and the live command list — **the authoritative version of this chapter.** |
| `/bug` | `/share` | Report a bug or share the conversation, after a consent screen. `/feedback` opens the same dialog for product feedback. |
| `/heapdump` | — | JS heap snapshot for high memory usage. **Contains your conversation and credentials** — do not share it. |

## Surfaces and integrations

| Command | What it does |
| --- | --- |
| `/teleport` /tp | Pull a Claude Code on the web session into this terminal — branch and conversation both. |
| `/remote-control` /rc | Expose this local session to claude.ai so you can drive it from another device. |
| `/desktop` /app | Continue in the Claude Code desktop app. |
| `/mobile` | QR code for the Claude mobile app. |
| `/ide` · `/chrome` | Manage IDE integrations; configure Claude in Chrome. |
| `/install-github-app` | Install the Claude GitHub App, optionally with Actions workflows and secrets. |
| `/install-slack-app` · `/web-setup` | Slack OAuth flow; connect GitHub to Claude Code on the web using your local `gh` credentials. |
| `/design-sync` /design-login | Convert your repo’s React design system and upload it so generated designs use your real components. |
| `/dataviz` | Chart and dashboard design guidance, with a bundled palette validator for contrast and colourblind safety. |
| `/voice [mode]` | Voice dictation: `hold`, `tap`, or `off`. |

The remainder are one-liners you will run once, if ever: `/login`, `/logout`, `/upgrade`, `/usage-credits`, `/passes`, `/privacy-settings`, `/remote-env`, `/setup-bedrock`, `/setup-vertex`, `/terminal-setup`, `/keybindings`, `/theme`, `/color`, `/scroll-speed`, `/powerup`, `/radio`, and `/stickers`.

> **Warning — Two commands that no longer exist**
>
> `/vim` was removed in v2.1.92 — toggle Vim editing from `/config` → **Editor mode**, or set `editorMode: "vim"` in settings. `/pr-comments` was removed in v2.1.91; ask Claude to read the PR comments, or use `/code-review --comment` to write them. Both still appear in guides written against older builds, which is a good reason to trust `/help` over any list, this one included.

> **Key — The eight that earn their keystrokes**
>
> `/context` tells you what is wrong · `/clear` fixes most of it · `/compact` when clearing is too blunt · `/rewind` undoes a bad turn · `/plan` before anything uncertain · `/goal` for unattended work · `/code-review` before every commit · `/doctor` when the harness itself is suspect.
>
> Three habits turn that list into a workflow. **Compact at around 60% of the window, not at 95%** — a summary written under pressure keeps less of what mattered. **Reach for `/rewind` instead of stacking “that didn’t work, try this instead”**, which leaves the failed attempt in context to be imitated. And **name every session** with `/rename`, so `/resume` is a list of workstreams rather than a wall of timestamps.
