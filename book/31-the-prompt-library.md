# The prompt library

_VI · Prompts · Chapter 31_

## Six patterns that do the work

| Pattern | Why it works |
| --- | --- |
| **One task per prompt** | Stack five requests and errors pile up across all five at once — and you cannot tell which instruction caused which failure. Ask, review, then ask the next. |
| **Be specific** | Layout, behaviour, content, constraints. The correction round costs more than the specificity would have. |
| **Show, don’t describe** | Paste files, attach mockups, link references. Material the agent can look at beats prose it has to interpret. “Match this file” resolves in one shot what a paragraph of stylistic adjectives will not. |
| **State the outcome, not the mistake** | “Use the real API response for every field” beats “don’t add placeholder data”. A model satisfies a negative instruction by avoiding the named failure, which can still miss the goal; naming the outcome closes the gap. |
| **“Act as” framing** | “Act as a security reviewer” shifts the _kind_ of scrutiny you get back from generic to role-specific. |
| **Ask it to think before it codes** | One sentence — “think through approaches before writing anything” — measurably improves complex-task output. |

Keep a running list of your agent’s repeat offences and convert each into a positive instruction. That list, not a generic style guide, is what belongs in `CLAUDE.md`.

## The scoped unit of work

The single most valuable block in this book. Paste it, fill four lines, send.

**P-01 · Scoped task · use for every feature**

```
Goal: <one feature, one sentence>
Touch only: <files or directories>
Do not touch: src/auth/**, src/billing/**, migrations/**
Done when: <named command exits 0, or a specific behaviour is observable>

Plan first. Do not edit anything until I approve the plan.
```

Every line earns its place. **Goal** stops scope creep. **Touch only** and **Do not touch** bound the blast radius — and the do-not-touch list is a security control, not a hygiene habit. **Done when** gives the loop a closing condition. The last line separates planning from execution.

## The loaded first turn

**P-02 · Loaded first turn · ground truth before reasoning**

```
!git log --oneline -10
!npm test 2>&1 | tail -20

Read @src/api/handlers/orders.ts and @src/api/schema.ts.
Don't read anything else yet.

Goal: add an idempotency key to POST /orders.
Touch only: src/api/handlers/orders.ts, src/api/schema.ts, tests/orders.test.ts
Do not touch: migrations/**, src/billing/**
Done when: `npm test tests/orders.test.ts` exits 0 and existing tests
still pass. Paste the test output.

Plan first.
```

## Interview me

**P-03 · Spec interview · before any large feature**

```
I want to build [brief description]. Interview me in detail using the
AskUserQuestion tool.

Ask about technical implementation, UI/UX, edge cases, concerns, and
tradeoffs. Don't ask obvious questions — dig into the hard parts I
might not have considered.

Keep interviewing until we've covered everything, then write a complete
spec to SPEC.md. The spec must name the files and interfaces involved,
state what is out of scope, and end with an end-to-end verification step.
```

## Test first, implementation second

**P-04 · Red · tests only**

```
Write tests for <function signature>.
Cases: <input → expected, one per line, including the error cases>

Do NOT implement it yet — tests only. Then run them and show me the
output. Confirm each failure is "not implemented", not a bug in the test.
```

**P-05 · Green · after committing the red state**

```
Now implement <function> until every test passes.
Do not modify the test file.
Run the suite after each change and paste the output.
```

## Debugging

**P-06 · First contact · rung one**

```
!<command that reproduces it> 2>&1 | tail -60

That's the failure after my last change. What's causing it?
Address the root cause — don't suppress the error.
```

**P-07 · Cause list · when a “fix” keeps failing**

```
Stop. Before writing any more code: list every possible cause of this
behaviour, ranked by likelihood, with how you'd rule each one out.

Do not propose a fix in this message.
```

**P-08 · Instrument · facts instead of guesses**

```
Add debug logging around <suspect path>, rerun <command>, and show me
the output. Don't fix anything yet.
```

## Review

**P-09 · Adversarial review · before you call it done**

```
Use a subagent to review the <feature> diff against <PLAN.md | the
requirements above>. Check that every requirement is implemented, the
listed edge cases have tests, and nothing outside the task's scope changed.

Report gaps, not style preferences. Flag only findings that affect
correctness or the stated requirements. Rank them: blockers, warnings, nits.
```

**P-10 · Cross-session review · fresh window, no history**

```
Review the change in this working directory. You are read-only: review and
discuss only — do not edit files, commit, push, or reset the tree.

Run `git rev-parse HEAD` and `git status` first, and anchor every finding to
that commit.

Look for edge cases, race conditions, data-durability gaps, injection, and
consistency with the existing patterns in @<sibling path>. Write findings as
a numbered list: file and line, what breaks, and the input or interleaving
that breaks it.

You did not write this code. Evaluate it on its own terms.
```

## Context control

**P-11 · Delegate the survey · keeps your window clean**

```
Use subagents to map <area>: the entry points, the lifecycle, where
state lives, and what the test strategy is.

Return a one-page summary with file paths. Do not paste file contents
into this conversation.
```

**P-12 · Steered compaction · before the window sprawls**

```
/compact keep only: the current task goal, files being edited, the test
command, and open decisions. Drop exploratory reads and superseded plans.
```

## Unattended

**P-13 · Reversibility gate · before you delegate anything consequential**

```
Before implementing, answer these and stop for my confirmation:

1. DOOR: Is this reversible? What exactly reverses it, and what does
   that cost — time, data loss, external effects?
2. BLAST RADIUS: What else breaks if this is wrong — this module,
   shared data, external consumers?
3. CONVERSION: If it is irreversible, what is the cheapest change that
   makes it reversible (branch, flag, down migration, dry run, backup,
   build-time check)? Propose it.
4. OPTIONS: At most three distinct approaches, then stop and recommend
   one. Do not enumerate further.
5. FALSIFIER: What single observation would show this was the wrong
   call? State it as something measurable.
6. If this is a one-way door: assume we shipped it and it failed badly.
   List the three most likely causes and what each costs to fix.

If every answer is "reversible, contained, cheap to undo", say so in
one line and proceed without waiting.
```

That last line carries as much weight as the questions. A gate that fires on every decision is the slow, deliberate process applied to work that never needed it, and an agent that stops to deliberate about a variable rename has cost you the advantage you delegated for. The prompt is the ergonomic layer, not the boundary — it runs the reasoning while you are watching, and it drifts like every other instruction. The deny and ask rules are what still hold when the session is long and nobody is reading the diff. → Ch. 05

**P-14 · Goal condition · all three parts present**

```
/goal `<named command>` exits 0 and `git status` is clean,
      <constraint: what must not change>,
      or stop after 20 turns
```

**P-15 · Headless fan-out unit · test on 3 files first**

```
Migrate $file from <old> to <new>. Run <test command for that file>.
Return exactly "OK" or "FAIL: <one-line reason>". Nothing else.
```

## Onboarding

**P-16 · Senior-engineer questions · ask directly, no framing needed**

```
How does <logging | auth | caching> work here? Show me the smallest
existing example.

What edge cases does <ClassName> handle?

Why does this call foo() instead of bar() on line <n>?

Look through <Symbol>'s git history and summarize how its API came to be.
```

## Harness maintenance

**P-17 · CLAUDE.md audit · run it quarterly**

```
Read @CLAUDE.md. For each line, answer: could a competent new contributor
infer this from the repo in under a minute?

List the lines that fail that test and say where each should move
instead — a path-scoped rule, a skill, a hook, or deleted. Don't edit
the file yet.
```

**P-18 · Write me a hook · Claude writes its own guardrails**

```
Write a PostToolUse hook that runs <formatter> after every Edit or Write,
and a PreToolUse hook that blocks any Bash command matching `rm -rf`
outside the project directory.

Put them in .claude/settings.json. Show me the file before writing it.
```

**P-19 · Turn a habit into a skill · after the third repeat**

```
I've pasted these same instructions three times now: [paste them]

Turn this into a skill at .claude/skills/<name>/SKILL.md.
Third-person description naming both what it does and when to use it.
Set disable-model-invocation: true — I want to trigger it by hand.
```

> **Key — Keep these as snippets, not as memory**
>
> The goal/touch/done-when shape works _because_ it is explicit every time. Keep it as a paste-in block — in a text expander, a scratch file, or a skill — instead of retyping it from memory, where details quietly drop. That is the whole argument for rule 10 of the field manual: **codify recurring prompts into reusable skills**, written once and invoked consistently instead of re-explained per session.
