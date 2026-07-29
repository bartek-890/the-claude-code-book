# The agentic loop

_I · Mechanism · Chapter 01_

Claude Code is an **agentic harness**: a runtime that gives a language model tools, an execution environment, and a policy for what enters its context. The model plans; the harness acts. Every task runs the same three-phase cycle, and every tool result feeds the next decision. That feedback _is_ the loop.

- **Agentic harness** — The runtime around a language model that supplies tools, context management, and an execution environment. Claude Code is one; so are Cursor, Codex, and Gemini CLI.
- **Agentic loop** — The cycle Claude Code runs on every task: **gather → act → verify**. Phases blend; a bug fix may cycle through all three dozens of times.
- **Turn** — One complete model response, including every tool call it makes, ending when Claude stops and hands control back to you (or to a Stop hook).

## The three phases

- **01 · Gather** — Read · Grep · Glob · WebFetch · shell reads
- **02 · Act** — Edit · Write · Bash · MCP tools
- **03 · Verify** — Tests · type errors · re-read · screenshot

| Phase | What Claude does | What it costs you |
| --- | --- | --- |
| **Gather** | Reads files, greps, globs, fetches docs, runs read-only commands | The dominant source of context growth. Every read stays in history for the rest of the session. |
| **Act** | Edits, writes, runs commands, calls MCP tools | The gated phase — most permission prompts live here. |
| **Verify** | Runs tests, reads type errors, re-reads the changed file, takes a screenshot | Nearly free, and the phase most sessions skip. Skipping it is why “done” is often wrong. |

A question about your codebase may only gather. A refactor cycles all three repeatedly. The practical consequence: **a task with no verify step has no closing condition.** Claude stops when the work _looks_ done, because “looks done” is the only signal available. You become the verification loop, and every mistake waits for you to notice it.

> **Key — The single highest-leverage habit in this book**
>
> Give Claude a check it can run: a test suite, a build exit code, a linter, a script that diffs output against a fixture, a browser screenshot compared to a design. Once a check exists, the loop closes on its own — Claude works, runs the check, reads the result, and iterates until it passes. Everything else in Part IV is elaboration on this one idea.

> **Note — One nuance that has changed under you**
>
> Giving Claude a check is not the same as _instructing_ it to verify. Opus 5 verifies its own work without being told. Anthropic’s guidance is now to **remove** the boilerplate you carried over from earlier models — “include a final verification step”, “use a subagent to verify”. On Opus 5 those lines cause over-verification. The check itself still earns its place: a command that returns pass or fail is what closes the loop. The instruction to go looking for one is what has become redundant. → Ch. 27

## Four ways to attach the check

They differ in how much setup they take and how hard they gate the stop.

| Mechanism | How it gates | Setup cost |
| --- | --- | --- |
| **In the prompt** | You ask Claude to run the check and iterate in the same message | Zero. Works on any task today. |
| `/goal` **condition** | A separate evaluator re-checks after every turn; Claude keeps working until it holds | One line. → Ch. 22 |
| **Stop hook** | Your script runs and blocks the turn from ending until it passes | A settings entry plus a script. → Ch. 14 |
| **Reviewer subagent** | A fresh model that never saw the reasoning tries to refute the result | An agent file. → Ch. 13, 23 |

> **Note — Stop hooks have a ceiling**
>
> Claude Code overrides a Stop hook and ends the turn after **8 consecutive blocks**. A check that can never pass will not trap the session forever — but it will burn eight turns finding that out. Make the check achievable.

## Show the evidence, don’t assert success

Ask Claude to surface the proof rather than the claim: the command it ran and what it returned, the test summary, the screenshot. Reviewing evidence is faster than re-running the verification yourself, and it is the only thing that works for a session you weren’t watching. This matters mechanically as well as psychologically — the `/goal` evaluator (→ Ch. 22) cannot call tools, so a result that never lands in the transcript may as well not exist.

Weak vs. strong task framing

— weak: no check, no closing condition > implement a function that validates email addresses — strong: the check is named, and running it is part of the task > write a validateEmail function. test cases: user@example.com true, "invalid" false, user@.com false. run the tests after implementing and paste the output.

## What extends the loop without replacing it

Four extension mechanisms layer on top. None of them changes the gather → act → verify cycle; each changes what is available inside it. Choosing between them is Part III’s job, but the one-line version:

- **SKILLS** — Package a workflow or reference; loads on demand → Ch. 12
- **SUBAGENTS** — Delegate to a separate context window → Ch. 13
- **HOOKS** — Run deterministic code at lifecycle events → Ch. 14
- **MCP SERVERS** — Connect external tools and data → Ch. 15

The rule of thumb that saves the most grief: **if it must happen every time with zero exceptions, it is a hook.** `CLAUDE.md` instructions and skills are advisory — Claude reads them and generally follows them. Hooks are code, and they run regardless of what the model decides.
