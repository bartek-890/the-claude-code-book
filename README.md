# The Claude Code Book

A working reference for Claude Code, the agentic coding harness — 31 chapters and 6 appendices on
what fills the context window, which files change the agent's behaviour, how to run a task you are
not watching, and what the bill actually responds to.

This repository is the Markdown edition, one file per chapter. A typeset PDF edition of the same
text is at [bartlomiejkrupa.dev/book](https://bartlomiejkrupa.dev/book).

**New here?** Start with [Start here](book/00-start-here.md), then
[Chapter 01 · The agentic loop](book/01-the-agentic-loop.md).
**In a hurry?** [Appendix A · The one-page cheatsheet](book/appendix-a-the-one-page-cheatsheet.md).

## What this covers

Claude Code is a harness, not a model: it supplies the tools, decides what enters the context
window, gates what the model may do, and decides when a turn ends. Most of what separates a good
session from a bad one lives in that layer, and most of it is configurable. The book is organised
around that claim — the mechanism first, then the controls, then the files that change behaviour,
then the procedures that use them.

It is written for an engineer who has already installed Claude Code, run a few sessions, and hit
the ceiling: the agent forgets instructions halfway through, ignores `CLAUDE.md`, burns tokens on
files nobody asked for, or declares work done that is not.

Behaviour was checked against **Claude Code v2.1.210–219, July 2026**, and version-gated facts
carry the release that introduced them. Where this book and your terminal disagree, your terminal
is right. Every claim's source is listed in [Appendix F](book/appendix-f-sources.md).

## Contents

### Part I · Mechanism

Five things happen on every task, in the same order, whether or not you know about them. Everything
in the rest of this book is a lever on one of the five. Learn the machine before you learn the
buttons.

- **01** · [The agentic loop](book/01-the-agentic-loop.md)
- **02** · [The startup tax: what loads before you type](book/02-the-startup-tax.md)
- **03** · [The context window: replay, decay, compaction](book/03-the-context-window.md)
- **04** · [Tools and the permission split](book/04-tools-and-the-permission-split.md)
- **05** · [Permission modes and the sandbox ladder](book/05-permission-modes-and-the-sandbox-ladder.md)

### Part II · Controls

The surface you actually touch: keys, prefixes, slash commands, and CLI flags. Most of this is
memorisable in an afternoon and pays back within a week. The four chapters are ordered by how often
you reach for them.

- **06** · [Keyboard shortcuts](book/06-keyboard-shortcuts.md)
- **07** · [The five prefixes](book/07-the-five-prefixes.md)
- **08** · [Slash commands: the complete map](book/08-slash-commands-the-complete-map.md)
- **09** · [The CLI: flags that change outcomes](book/09-the-cli.md)

### Part III · Harness

An agent harness is everything around the model: rule files, permissions, subagents, hooks,
verification. It decides more of the outcome than the model does. Eight chapters, one per
mechanism, each answering the same two questions — what does this actually change, and when should
I reach for it instead of the others?

- **10** · [CLAUDE.md: why agents ignore it](book/10-claude-md.md)
- **11** · [Rules, imports, and auto memory](book/11-rules-imports-and-auto-memory.md)
- **12** · [Skills](book/12-skills.md)
- **13** · [Subagents](book/13-subagents.md)
- **14** · [Hooks](book/14-hooks.md)
- **15** · [MCP](book/15-mcp.md)
- **16** · [Plugins](book/16-plugins.md)
- **17** · [settings.json](book/17-settings-json.md)

### Part IV · Flows

Nine procedures, each written to be followed with the book open beside the terminal. They share one
shape — bound the work, give Claude a check it can run, review the evidence, checkpoint — and
differ in how much of the session you intend to watch.

- **18** · [Explore → Plan → Implement → Commit](book/18-explore-plan-implement-commit.md)
- **19** · [Spec-first: let Claude interview you](book/19-spec-first-let-claude-interview-you.md)
- **20** · [Test-driven: the red-green loop](book/20-test-driven-the-red-green-loop.md)
- **21** · [The debugging escalation ladder](book/21-the-debugging-escalation-ladder.md)
- **22** · [The unattended run](book/22-the-unattended-run.md)
- **23** · [Parallel sessions: worktrees and writer/reviewer](book/23-parallel-sessions.md)
- **24** · [Fan-out: one migration, two thousand files](book/24-fan-out.md)
- **25** · [Onboarding to an unfamiliar codebase](book/25-onboarding-to-an-unfamiliar-codebase.md)
- **26** · [Recovery: when it has gone wrong](book/26-recovery.md)

### Part V · Economics

Three independent dials sit under every session: which model runs, how many tokens it spends, and
how many of those tokens you pay full price for. Most teams turn one and assume they have
optimised. They compose — and one of them, counter-intuitively, gets worse the harder you turn it.
A fourth chapter covers the part everyone skips: proving the cheaper path still ships correct work.

- **27** · [Model tiers: pick by role, not benchmark](book/27-model-tiers.md)
- **28** · [Effort, thinking, and fast mode](book/28-effort-thinking-and-fast-mode.md)
- **29** · [The cost levers that actually move the bill](book/29-the-cost-levers.md)
- **30** · [Evals: keeping the cuts honest](book/30-evals.md)

### Part VI · Prompts

Prompts you paste, not prompts you remember. The shapes work because they are explicit every time —
a saved snippet keeps its details, and a remembered one quietly loses them.

- **31** · [The prompt library](book/31-the-prompt-library.md)

### Appendices

The parts you will come back to: one page to print, two skeletons to copy, a symptom table to scan
when something is wrong, a glossary, and where every fact came from.

- **A** · [The one-page cheatsheet](book/appendix-a-the-one-page-cheatsheet.md)
- **B** · [The lean CLAUDE.md skeleton](book/appendix-b-the-lean-claude-md-skeleton.md)
- **C** · [The ten-file harness](book/appendix-c-the-ten-file-harness.md)
- **D** · [Symptom → cause → fix](book/appendix-d-symptom-cause-fix.md)
- **E** · [Glossary](book/appendix-e-glossary.md)
- **F** · [Sources](book/appendix-f-sources.md)

### End matter

- [About this book](book/99-about-this-book.md)
- [Last thing](book/98-last-thing.md)

## Other editions

| Edition | Where |
| --- | --- |
| Markdown, one file per chapter | this repository |
| Typeset PDF | [bartlomiejkrupa.dev/book](https://bartlomiejkrupa.dev/book) |
| Typeset PDF, pay what you want | [Gumroad](https://book.bartlomiejkrupa.dev/l/the-claude-book) |

Future editions are included in all three: the same links serve the current file.

## Corrections

Claude Code ships several releases a week, so parts of this will go stale. If you find something
wrong, or something that used to be true, open an issue or write to **bartkru@pm.me**. Corrections
are credited in the edition that carries them.

The Markdown under `book/` is exported from the book's typesetting source, so a pull request
against those files would be overwritten by the next export. An issue is the durable path.

## About

Written by [Bartłomiej Krupa](https://bartlomiejkrupa.dev), who builds and ships software with
coding agents and writes about the part nobody puts in the demo: what fills the context window,
what the bill responds to, and why a session that looked fine produced work that was not.

Claude and Claude Code are products of Anthropic PBC. This book is independent — unaffiliated,
unendorsed, and unreviewed by Anthropic. It is one engineer's reading of the public documentation
plus his own measured practice.

## Licence

© 2026 Bartłomiej Krupa. Licensed under
[Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International](LICENSE)
(CC BY-NC-ND 4.0).

Read it, print it, quote it with attribution, and link it anywhere. Do not sell it or publish an
altered version. Quoting a passage in an article, a talk, or a README is exactly what the licence
is for.
