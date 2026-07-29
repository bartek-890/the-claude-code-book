# Subagents

_III · Harness · Chapter 13_

A subagent runs in its **own context window** and returns only a summary. The heavy intermediate tokens — large file reads, MCP documentation, search output — never enter your conversation. That is the entire value proposition, and it is worth more than it sounds: context is your fundamental constraint, so anything that spends someone else’s window is close to free.

## Built-in subagents

| Agent | Model / tools | Purpose |
| --- | --- | --- |
| `Explore` | Inherits the session model, capped at Opus on the Claude API. Read-only — Write and Edit denied. | File discovery, code search, codebase exploration. Claude specifies a thoroughness level: **quick**, **medium**, or **very thorough**. |
| `Plan` | Inherits the session model. Read-only. | Codebase research during plan mode, so exploration output stays out of the planning window. |
| `general-purpose` | Full tool access | Multi-step tasks and searches where you are not confident of a first-try hit. Resumable, unlike the two above. |

> **Note — Two exceptions worth remembering**
>
> `Explore` and `Plan` are the **only** subagents that skip your `CLAUDE.md` files and the parent session’s git status — deliberately, to keep research fast and cheap. There is no setting to change that. If a rule must reach them (“ignore `vendor/`“), restate it in the delegation prompt. Both are also one-shot and return no agent ID, so they cannot be resumed.
>
> A user or project subagent _named_ `Explore` overrides the built-in and keeps its own `model` field — define one with `model: haiku` to force cheap exploration.

## Writing your own

Files live in `.claude/agents/` (project) or `~/.claude/agents/` (personal). Claude Code watches both and picks up changes within seconds.

```
---
name: code-reviewer
description: Reviews a diff in a fresh context after implementation, before commit
             or PR. Read-only. Use proactively once a change is complete.
tools: Read, Grep, Glob, Bash
model: sonnet
effort: high
color: red
---

You are a senior engineer reviewing a diff you did not write.

Report only findings that affect correctness or the stated requirements.
Rank them: blockers, warnings, nits. For each, give the file and line,
the failure scenario, and the smallest fix.

Do not comment on style the formatter already enforces.
```

## Frontmatter reference

| Field | What it does |
| --- | --- |
| `name` req | Lowercase letters and hyphens. Hooks receive it as `agent_type`. The filename need not match. |
| `description` req | When Claude should delegate here. **This is the whole automatic-selection mechanism** — write it as a trigger, not a title. |
| `tools` | Allowed tools. Inherits everything available to subagents if omitted. If no entry resolves to a real tool, the subagent usually fails to start. |
| `disallowedTools` | Removed from the inherited or specified list |
| `model` | `sonnet` · `opus` · `haiku` · `fable` · full ID · `inherit` (default) |
| `effort` | Overrides the session level while active. **Low effort is Anthropic’s documented use case for subagents.** |
| `permissionMode` | `default` · `acceptEdits` · `auto` · `dontAsk` · `bypassPermissions` · `plan` |
| `maxTurns` | Hard cap before the subagent stops |
| `skills` | Skills preloaded at startup — **full content, not just the description** |
| `mcpServers` | Scope MCP servers to this agent, by name or inline definition |
| `hooks` | Lifecycle hooks scoped to this subagent. Ignored for plugin subagents. |
| `memory` | `user` · `project` · `local` — its own persistent memory directory, separate from the main conversation’s |
| `background` | `true` always runs it as a background task, even when Claude wants the result immediately |
| `isolation: worktree` | Run in a temporary git worktree — an isolated copy of the repo |
| `color` | Display colour in the task list and transcript |
| `initialPrompt` | Auto-submitted first turn when the agent runs as the _main_ session agent via `--agent` |

## What a subagent sees — and what it never sees

| Loaded at startup | Never reaches it |
| --- | --- |
| Its own system prompt (the markdown body) plus environment details | Your conversation history |
| The delegation message Claude writes | The main conversation’s auto memory |
| Every level of the `CLAUDE.md` hierarchy (except Explore/Plan) | Your output style — it runs its own system prompt |
| Git status snapshot from the parent session | Skills you already invoked in the main window |
| Full content of skills named in `skills:` | Files Claude already read |
| A sibling roster of other named agents v2.1.206+ | The parent’s context-window size — it gets its own model’s |

## Foreground, background, and resuming

Claude chooses foreground or background unless `background: true` forces the latter. `/tasks` lists what is running; `Ctrl+X Ctrl+K` twice stops all of them. Each invocation creates a fresh instance — but a completed subagent can be **resumed with its full history**, including previous tool calls and reasoning, by asking Claude to continue that agent’s work. Under the hood Claude uses the `SendMessage` tool with the agent’s ID or name.

```
> Use the code-reviewer subagent to review the authentication module
[agent completes]
> Continue that code review and now analyze the authorization logic
[Claude resumes the same subagent with full prior context]
```

Transcripts live in `~/.claude/projects/{project}/{sessionId}/subagents/`, persist independently of the main conversation (compaction does not touch them), and are cleaned up on the `cleanupPeriodDays` schedule.

> **Warning — Subagent edits are outside your checkpoints**
>
> Except for a foreground forked skill, edits a subagent makes **are not restored by `/rewind`** — including a background `/code-review --fix` run. Use git to revert those. This is the strongest single argument for committing before you delegate anything that writes.

## Fork: the one subagent that inherits everything

A **fork** copies the parent conversation and system prompt into the subagent, which makes it the exception to almost every rule above: it sees your history, your output style, and your auto memory. Use it when the task needs the context you have already built; use a named subagent when the task benefits from _not_ having it.

## Four patterns that pay

1. **Isolate high-volume reads**

   Log triage, multi-file surveys, reading an MCP server’s API reference. In one demonstration, an API planner reading Stripe’s docs through an MCP docs server consumed **~54K tokens** — illustrative of the scale. In a subagent, your window pays for the conclusion.

2. **Run parallel research**

   Two or three independent questions, fanned out at once, each returning a summary. Cheapest on Haiku, which is built for exactly this. → Ch. 27

3. **Adversarial review in a fresh context**

   The reviewer never saw the reasoning that produced the change, so it evaluates the result on its own terms. This is the most reliable quality gate available inside one session.

4. **Chain them**

   Research agent → implementation in the main window → review agent. Each stage’s noise stays in its own window; only the conclusions cross.

> **Warning — The reviewer’s bias, and how to blunt it**
>
> A reviewer prompted to find gaps **will** report some, even when the work is sound — that is what it was asked to do. Chasing every finding produces over-engineering: extra abstraction, defensive code, tests for cases that cannot happen. Tell the reviewer to flag only what affects correctness or the stated requirements, and treat the rest as optional.

## Writing a description Claude will actually act on

The `description` is the automatic-delegation mechanism. Compare:

| Weak — a title | Strong — a trigger |
| --- | --- |
| “Code reviewer” | “Reviews a diff or files in a fresh context. Use after completing an implementation, before commit or PR. Read-only.” |
| “Explores the codebase” | “Read-only recon: searches code, reads logs, runs diagnostic commands. Returns only a compact summary, so verbose output stays out of the main context. Use proactively for any task producing lots of output.” |

The strong versions name the _situation_, not the capability — and they say “proactively” where you want Claude to reach for it without being told. You can also always force it explicitly: _“use a subagent to review this code for edge cases.”_
