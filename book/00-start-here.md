# Start here


This is a book about a **harness**, not a model. Claude Code is the runtime around the model: it supplies tools, decides what enters the context window, gates what the model may do, and decides when a turn ends. Almost everything that separates a good session from a bad one lives in that layer — and almost all of it is configurable.

## Who this is written for

An engineer who has already installed Claude Code, run a few sessions, and hit the ceiling: the agent forgets instructions halfway through, ignores `CLAUDE.md`, burns tokens on files nobody asked for, or declares work “done” that isn’t. You do not need to be an AI researcher. You do need to be comfortable in a terminal and to have opinions about your own codebase.

If you have never opened Claude Code, read Part I and Chapter 18, run one session, then come back. The rest of the book will land better with one real session behind it.

## What you should be able to do afterwards

- Explain, precisely, where every token in your context window came from — and remove the ones you did not want.
- Write a `CLAUDE.md` the agent actually follows, and know what to move out of it.
- Pick the right extension mechanism — skill, subagent, hook, MCP server — instead of reaching for whichever one you learned first.
- Run an unattended multi-turn task that stops on a machine-checked condition, not a vibe.
- Recover a session that has gone wrong in under thirty seconds.

## How the book is organised

- **PART I — MECHANISM** — How the loop and the window actually behave
- **PART II — CONTROLS** — Keys, prefixes, slash commands, CLI flags
- **PART III — HARNESS** — Files that change behaviour: rules, skills, agents, hooks
- **PART IV — FLOWS** — Nine step-by-step procedures, start to finish
- **PART V — ECONOMICS** — Tier, effort, the levers on the bill, and how to score the result
- **PART VI — PROMPTS** — A library you paste from, not retype
- **APPENDICES** — Cheatsheet, skeletons, failure table, glossary, sources

Parts I–III are reference: read once, return often. Part IV is procedural — each flow is a numbered sequence you can follow with the book open beside the terminal. Parts V and VI are optimisation: they only pay off once the earlier parts are habit.

## Conventions used throughout

| Element | Means |
| --- | --- |
| TAG | A label — tool name, version gate, or classification |
| Ink-filled block | Terminal output or a code block you can copy |
| Bordered **PROMPT** block | Text to paste into Claude Code verbatim |
| Bordered box, red edge | A failure mode that costs real time |
| `inline mono` | An exact string: command, flag, filename, or setting key |
| → Ch. 12 | Cross-reference to another chapter in this book |

> **Warning — Version note — read this one**
>
> Claude Code ships multiple releases per week. Every version-gated fact in this book is marked with the version that introduced it (for example, v2.1.187+). Behaviour described without a gate was current in **v2.1.210–220, August 2026**. Two things age fastest: model names with prices, and any count of built-in tools or commands. When a number in this book disagrees with `/help`, `/context`, or `claude --help` on your machine, your machine is right.

> **Note — A word on where the facts come from**
>
> Three sources, and it is worth knowing which is which.
>
> - **Anthropic’s documentation** supplies the mechanics — tool behaviour, hook contracts, frontmatter fields, flags.
> - **Measured practice** supplies the thresholds: Anthropic’s own engineering write-ups plus published practitioner numbers. The ~84% drop in permission prompts from sandboxing. The 150–200 instruction adherence ceiling. The 85% cut in upfront tool tokens Anthropic measured for deferred tool loading on the API — the same pattern Claude Code applies to MCP.
> - **Judgement** supplies the ordering — which lever to pull first.
>
> Appendix F opens with those three numbers, each with its source, what it actually measured, and how far it travels — two are Anthropic’s own published figures, the third is a practitioner rule of thumb and is labelled as one. The rest of the appendix lists everything else. Judgement is mine, and where I am less sure I say so.
