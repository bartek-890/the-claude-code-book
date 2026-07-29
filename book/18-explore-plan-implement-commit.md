# Explore → Plan → Implement → Commit

_IV · Flows · Chapter 18_

The default flow, and the one to reach for when you are not sure which flow applies. It exists to stop Claude solving the wrong problem — which is the most expensive failure mode available, because the code is _good_, it just does the wrong thing.

- **Plan mode** — A permission mode in which Claude reads, searches, and plans but makes no edits until you approve. `Shift+Tab` mid-session, or `claude --permission-mode plan` at startup, or `/plan [task]`.

## The four phases

1. **Explore — in plan mode, read only**

   Point Claude at the relevant code and let it read. Name the files; do not make it hunt.

   ```
   read src/auth and explain how we handle sessions and login.
   also check how we load secrets from env.
   ```

2. **Plan — still in plan mode**

   Ask for a concrete implementation plan: which files change, what the flow is, what the risks are. Then **press `Ctrl+G`** and edit the plan in your editor.

   ```
   I want to add Google OAuth. What files change? What's the session
   flow? Write a plan.
   ```

   Correcting the spec costs one line. Correcting the diff costs an afternoon.

3. **Implement — exit plan mode**

   Let Claude code against the approved plan, and name the verification in the same breath.

   ```
   implement the OAuth flow from your plan. write tests for the
   callback handler, run the suite, and fix any failures.
   ```

4. **Review, then commit**

   Scan the diff for the four changes most likely to be silently wrong — **file deletions, changed API signatures, new dependencies, schema edits** — then run `/code-review` and commit on green.

   ```
   commit with a descriptive message and open a PR
   ```

## When to skip the plan

The rule of thumb: **if you could describe the diff in one sentence, skip the plan.**

| Use plan mode | Skip it |
| --- | --- |
| The approach is uncertain | Fixing a typo or adding a log line |
| The change spans multiple files | Renaming a single variable |
| You are unfamiliar with the code | Scope is clear and the fix is small |

## What plan mode actually enforces

Plan mode _feels_ like the tools are hard-locked. They are not. Reverse-engineering of the harness found plan mode is, in Armin Ronacher’s words, “just a rather short predefined prompt” — the tools “turn into read-only” by instruction, not by technical restriction. The write tool stays available; the agent uses it to edit exactly one file while “read-only”: the plan itself.

- **The plan is a file.** Markdown in Claude Code’s plans folder. `Ctrl+G` opens it in `$EDITOR`.
- **The injected prompt has its own four phases**, distinct from the user-facing ones: _Initial Understanding_ → _Design_ → _Review_ → _Final Plan_.
- **Exit reads the plan back** from disk and starts executing against it. If you were only exploring, there is nothing to execute and the exit step has nothing to do.

> **Warning — The read-only promise is behavioural, not sandboxed**
>
> If you need a hard guarantee that nothing is written — an untrusted repo, a destructive tool — reach for OS-level sandboxing or permission rules, not plan mode alone. → Ch. 05

## The DIY alternative

Because the mechanism is a prompt plus a markdown file, you can approximate it without switching modes: iterate on a visible markdown file on disk, asking Claude clarifying questions until the plan is solid.

| Aspect | Plan mode | DIY markdown handoff |
| --- | --- | --- |
| Enforcement | Tools flip read-only by prompt; exit gates on approval | Normal tools stay live; your discipline gates edits |
| Artifact | Markdown in a hidden plans folder; edit via `Ctrl+G` | A visible file you own and version |
| Approval | Built-in approve / reject UI on exit | You read and edit the file yourself |
| Best for | The standard flow — one keystroke, built-in gate | Full transparency; a plan that outlives the session |

The only thing DIY cannot cheaply replicate is the approval UI. Everything else — the plan, the phased prompt — is reproducible with a file and a good prompt. Chapter 19 is the DIY route taken seriously.
