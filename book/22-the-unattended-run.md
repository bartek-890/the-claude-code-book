# The unattended run

_IV · Flows · Chapter 22_

`/goal` sets a completion condition and Claude keeps taking turns toward it — no per-turn prompting — until a **separate** small fast model (Haiku by default) confirms the condition is met. Paired with auto mode, the whole run is hands-off. The only real skill is writing a condition whose proof Claude actually surfaces.

There are two ways to run an agent, and this chapter is about the second one. You can sit on top of it and drive every step, which beats typing the code yourself but leaves you babysitting: each time it pauses you reload the whole problem into your head, and if you look away and lose the thread it ends up slower than doing the work. Or you treat it the way you would treat someone who reports to you — say what you want and roughly how, then come back when there is something worth looking at. **Neither mode changes who is accountable.** Once you hand real work to an agent you stop being the person who wrote the code and become the person answerable for it, and when something breaks in production nobody asks how the code got written. The trade is a good one, but it moves your job from authoring to reviewing, and the review is not optional: left unwatched and unchecked, an agent produces slop — not because it is stupid, but because that is what anything produces when nobody checks its work.

- **Completion condition** — The plain-text end state Claude works toward, up to **4,000 characters**. Setting it starts a turn immediately, with the condition itself as the directive.
- **Evaluator** — The small fast model that, after each turn, reads the condition and the conversation so far and returns yes/no plus a short reason. **It does not call tools.**

Requires v2.1.139+, one active goal per session. A new goal replaces the old one.

## The flow

1. **Write a condition with all three parts**

   | Part | What it is | Example |
   | --- | --- | --- |
   | Measurable end state | One checkable fact | a test result, a build exit code, an empty queue |
   | Stated check | How Claude proves it | “`npm test` exits 0”, “`git status` is clean” |
   | Constraints | What must not change on the way | “no other test file is modified” |

   ```
   /goal `npm test test/auth` exits 0 and the lint step is clean,
         no other test file is modified, or stop after 20 turns
   ```

2. **Bound the run**

   Always add `or stop after N turns` or `or stop after N minutes`. Claude reports progress against it each turn and the evaluator judges that from the conversation too.

3. **Turn on auto mode so tool calls don’t block**

   The two compose exactly: auto mode removes per-_tool_ prompts, `/goal` removes per-_turn_ prompts. Together they make an unattended run possible; either alone does not.

4. **Check status while it runs**

   `/goal` with no argument shows the condition, elapsed time, turns evaluated, token spend, and **the evaluator’s most recent reason** — which is the single most useful diagnostic when a goal is not converging.

5. **Clear or finish**

   `/goal clear` (aliases: `stop`, `off`, `reset`, `none`, `cancel`). `/clear` also removes it. On success the goal clears and an achieved entry is recorded in the transcript.

> **Key — The failure mode, and it is the only one that matters**
>
> The evaluator **cannot run your tests, read a file, or check `git status`**. It judges the condition against what Claude has already surfaced in the conversation. If Claude did not run the check and echo the result into the transcript, the evaluator is judging on absent evidence — and you get either a goal that never completes, or a “yes” the work did not earn.
>
> So do three things. **Name the command in the condition** — “`npm test` exits 0”, not “tests pass”. Make Claude run it every turn. And take proof a machine can check — an exit code, a passing summary line, a clean `git status` — over prose the model can assert for free.

## Verifiable versus vague

| Vague — spins | Verifiable — terminates |
| --- | --- |
| “Fix the auth bug” | “`npm test test/auth` exits 0 and no other test file is modified” |
| “Clean up this file” | “`user.ts` is split so each module is under 200 lines and the build passes” |
| “Handle the backlog” | “Every issue labeled `migrate` is closed and the queue is empty” |
| “Make the code better” | — not a completion condition at all. Do not start. |

## Four ways to keep a session moving

| Approach | Next turn starts when | Stops when |
| --- | --- | --- |
| `/goal` | The previous turn finishes | A model confirms the condition is met |
| `/loop` | A time interval elapses (or Claude self-paces) | You stop it, or Claude decides work is done |
| Stop hook | The previous turn finishes | Your own script or prompt decides |
| Auto mode | N/A — operates _within_ a turn | Claude judges the work done |

`/goal` and a Stop hook both fire after every turn. `/goal` is the session-scoped, type-a-condition version; a Stop hook lives in settings, applies to every session in scope, and can run a script for deterministic checks. Prefer the hook when the check is the same every time; prefer `/goal` when the condition is this task’s.

## Headless

```
claude -p "/goal CHANGELOG.md has an entry for every PR merged this week"
```

The whole loop runs in one non-interactive invocation. `Ctrl+C` stops it early. Resume behaviour: a goal still active when a session ends is restored on `--resume` or `--continue` — the condition carries over, but the turn count, timer, and token-spend baseline all reset. An achieved or cleared goal is not restored.

## When it will not start

Three reasons, and the command tells you which: the workspace trust dialog has not been accepted (the evaluator is part of the hooks system), `disableAllHooks` is set at any settings level, or `allowManagedHooksOnly` is set in managed settings.

## Which doors the run can reach

To a model, `git mv` and `DROP TABLE` are the same shape of token sequence. It executes both with the same confidence and the same latency, and explains both equally well afterwards — including, sometimes, explaining incorrectly which one it just did. Nothing in the output distinguishes the reversible from the permanent, so the distinction has to be made before the run starts. Amazon’s 2015 shareholder letter has the vocabulary for it: a **two-way door** is a decision you can reopen and walk back through, and a **one-way door** is one you cannot. Two-way doors go to the agent immediately. One-way doors stop here.

Do not try to route on speed. Whether delegating a given task will be faster is genuinely hard to know in advance and your own sense of it is unreliable — the developers in METR’s 2025 trial came out 19% slower on real issues in their own repositories while estimating they had been 20% faster. What you _can_ know before writing the prompt is what the change costs if it is wrong. Route on that.

| The task | Reverses with | Route |
| --- | --- | --- |
| Rename, restructure, refactor behind an existing interface | git | Delegate now |
| Pick a library for one internal module | An afternoon | Delegate now |
| Add a nullable column and backfill it | A down script | Delegate — with the down script written first |
| Drop a column, delete records | A restore you have actually run | Back up, then run it yourself |
| Publish a package, an API, a CLI flag | Nothing | Decide it yourself; delegate the code |
| Anything that sends, publishes, pays, or deletes | Nothing | Never unattended |

Then convert rather than deliberate. A branch or worktree, a feature flag, a down migration written first, a `--dry-run` pass, a build-time check that fails loudly, a restore you have actually performed — each one moves a task out of the slow column. Twenty minutes spent making a decision reversible beats two hours spent deciding whether it is right, and note what makes a backup a conversion: a restore you have run. An untested backup is a belief about reversibility.

**A classification the model is merely told about is not a classification.** In July 2025 a Replit agent deleted a production database during an explicit code freeze, after being told not to — in the customer’s words — “eleven times in ALL CAPS”, and then reported that the deletion could not be undone. That second claim was false too; the rollback worked. What Replit shipped afterwards was not a stronger system prompt but automatic separation of development and production databases. Put the classification in `permissions.deny` and `permissions.ask`, where the mode cannot override it, and keep production credentials out of the environment the agent runs in. → Ch. 05

> **Warning — Before you walk away**
>
> An unattended run has your credentials and your filesystem. Bound it on **five** axes, not one: a verifiable condition, a turn cap, a spend cap (`--max-budget-usd`), an explicit `--allowedTools` list, and an isolation tier appropriate to the blast radius. And commit first — subagent edits are outside `/rewind`. → Ch. 05, 09
