# Spec-first: let Claude interview you

_IV · Flows · Chapter 19_

For anything larger than a single feature, the highest-leverage move is to spend a session producing **no code at all** — just a spec. The spec is the durable artifact; the sessions that consume it are disposable. It also survives compaction, session resets, and your own memory, which none of the alternatives do.

People who switch from prompting one step at a time to handing over a complete design document describe the same discontinuity, and they do not describe it as an improvement in prompting: the same feature comes back at a level of quality that makes the two approaches feel like two different tools. The mechanism is Chapter 3’s. A complete first turn puts every constraint in the window _before_ there is any code to contradict them, near the top edge where recall is strongest, and in a file that gets re-read rather than a message that gets summarised. Drip-feeding does the opposite of all three.

## The flow

1. **Open a session whose only job is the spec**

   Plan mode, or just a fresh window. You are not going to write code here, so nothing needs to be unlocked.

2. **Ask to be interviewed, not answered**

   The point is to surface the decisions you have not made yet.

   **Prompt · The interview · replace the bracket**

   ```
   I want to build [brief description]. Interview me in detail using the
   AskUserQuestion tool.

   Ask about technical implementation, UI/UX, edge cases, concerns, and
   tradeoffs. Don't ask obvious questions — dig into the hard parts I
   might not have considered.

   Keep interviewing until we've covered everything, then write a
   complete spec to SPEC.md.
   ```

3. **Answer honestly, including “I don’t know”**

   “I don’t know, what would you recommend and why?” is a legitimate answer and often the most productive one. The interview is where the cheap decisions get made.

4. **Read `SPEC.md` and fix it yourself**

   A useful spec is **self-contained**: it names the files and interfaces involved, states what is out of scope, and ends with an end-to-end verification step that proves the feature works. Time spent making it precise pays back more than time spent watching the implementation.

5. **Start a fresh session to execute it**

   `/clear`, or a new window. The new session has clean context focused entirely on implementation, and a written reference it can re-read at any point — unlike a conversation, which compaction will eventually summarise away.

   ```
   Read @SPEC.md. Implement phase 1 only: the data model and the
   `add` command. Do not start phase 2.
   Done when: `npm test` exits 0 and `npm run typecheck` is clean.
   ```

## Phase the build before you prompt it

Five decisions belong in the spec, and all five are cheaper to make before the first line of code exists:

| Decision | The rule |
| --- | --- |
| **Define the MVP** | The simplest version that works. For a habit tracker: add a habit, check in, see a streak. Accounts are phase two. Agents perform measurably better on small focused asks. |
| **Pick a boring stack** | React, Next.js, Tailwind, Supabase, Python. The model has seen enormous amounts of these, so output quality is higher and error rates lower. Boring is a feature. |
| **Use the official generator** | `npm create vite@latest`, `npx create-next-app`, `rails new`. The generator tracks current conventions; an agent’s training data is a snapshot that was already stale the day a major version shipped. |
| **Generate the design system first** | If there is a UI, produce the design artifact _before_ the coding agent runs — a file it loads, not a screenshot you describe. |
| **Keep the always-on config lean** | The spec is per-session context; `CLAUDE.md` loads every turn and gets the opposite treatment. → Ch. 10 |

> **Key — Rent versus purchase**
>
> Front-loading context on a complex task does not contradict the lean-config rule. The always-on file stays minimal **because it loads every turn** — that is rent. Per-task context should be as complete as the task demands — that is a one-time purchase. Confusing the two is why people either starve their agent or drown it.
