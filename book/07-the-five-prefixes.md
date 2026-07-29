# The five prefixes

_II · Controls · Chapter 07_

The first character of your input changes what the whole line means. Five of them matter, and two — `!` and `@` — change how much of the codebase Claude has to read to answer you, which makes them context-engineering tools disguised as typing conveniences.

## `/` — commands and skills

Start with `/` to run a built-in command, a bundled skill, a skill you wrote, or one a plugin contributed. Type `/` alone for the full menu, or `/` plus letters to filter. There is no distinction in the menu between a command and a skill; both are things you invoke by name. → Ch. 08, 12

## `!` — shell mode

Prefix with `!` and the rest of the line runs in _your_ shell. The output is added to the session and Claude responds to it.

```
!git status
!npm test 2>&1 | tail -30
!gh pr view 412 --json title,body
```

This is the highest-signal, lowest-token way to give Claude facts. Compare: asking “what’s the current git state?” makes Claude spend a tool call and a permission prompt; typing `!git status` puts the exact answer in context for the price of the output itself. Use it for anything where _you_ already know the command.

> **Note — Related: background shell jobs**
>
> Long-running commands can be backgrounded with `Ctrl+B` so the session stays responsive, and `/tasks` lists what is running — shells and subagents both.

## `@` — file mentions

Type `@` for path autocomplete. The referenced file is read before Claude responds, which means you have replaced a search with a fact.

```
read @src/auth/session.ts and @src/auth/tokens.ts, then explain
how refresh works. don't read anything else yet.
```

Two habits that compound: use tab-completion rather than typing paths from memory (a wrong path costs a failed read plus a search), and add “don’t read anything else yet” when you want a tight first turn.

## `\` + `Enter` — the portable newline

Works in every terminal, needs no configuration. See Chapter 6 for the four alternatives.

## `:` — emoji shortcodes v2.1.217+

Type a full `:name:` to insert the emoji, or two or more characters for suggestions. Cosmetic, but it stops you leaving the terminal to find one.

## The composite move

The prefixes are most valuable stacked into a single first message. This is the shape of a first turn that does not waste a hundred tool calls:

**Prompt · Loaded first turn · paste as one message**

```
!git log --oneline -10
!npm test 2>&1 | tail -20

Read @src/api/handlers/orders.ts and @src/api/schema.ts.

Goal: add an idempotency key to POST /orders.
Touch only: src/api/handlers/orders.ts, src/api/schema.ts, tests/orders.test.ts
Do not touch: migrations/**, src/billing/**
Done when: `npm test tests/orders.test.ts` exits 0 and the existing
tests still pass. Paste the test output.

Plan first — do not edit anything until I approve.
```

Every element earns its place: two shell commands establish ground truth, two file mentions replace an exploratory search, an explicit touch/do-not-touch list bounds the blast radius, and a named check closes the loop. This block reappears in Chapter 31 as a reusable template.
