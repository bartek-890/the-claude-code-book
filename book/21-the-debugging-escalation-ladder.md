# The debugging escalation ladder

_IV · Flows · Chapter 21_

Six rungs, in order. Each is cheaper than the one after it, so climb rather than jump. The single most important rung is the fifth, and it is the one nobody uses in time.

1. **Paste the full error verbatim**

   Plus one line on what you were doing. **Resist the urge to summarise** — the stack trace _is_ the context, and the frame you would have trimmed is usually the one that matters. Most of the time this is the whole fix.

   ```
   !npm test 2>&1 | tail -60

   that's the failure after my last change. what's causing it?
   ```

2. **Still failing? Ask for logs, then paste the output**

   Logs replace the agent’s guesses about runtime state with facts. This converts a reasoning problem into a reading problem.

   ```
   add debug logging around the token refresh path, rerun the failing
   test, and show me the output. don't fix anything yet.
   ```

3. **Bug keeps returning after “fixes”? Demand a cause list first**

   This forces diagnosis over symptom-patching and usually surfaces the real root cause on the first pass.

   ```
   before writing any more code: list every possible cause of this
   behaviour, ranked by likelihood, with how you'd rule each one out.
   ```

4. **Once understood, write the breaking test first**

   A test that fails _because of the bug_. Then fix until it passes. The fix is now proven, and the bug cannot return silently.

5. **Three failed prompts on the same problem? Stop.**

   Start a fresh session and rephrase from scratch, incorporating what you learned. A conversation that has gone wrong **keeps contaminating further attempts** — the model anchors on its own earlier mistakes. A clean slate beats a rescue almost every time, and the git checkpoint means quitting costs nothing.

6. **Give the agent real eyes**

   For anything involving a UI, an MCP browser tool (Playwright, or Claude’s Chrome integration) lets the agent drive your app and _observe_ behaviour instead of inferring it from source. Direct observation shortens every UI debugging loop.

> **Key — The two-correction rule**
>
> Anthropic’s own version of rung five is stricter: **after two failed corrections** on the same issue, the context is polluted with failed approaches. `/clear` and write a better initial prompt incorporating what you learned. The instinct to push on for “just one more try” is exactly wrong — each attempt makes the next one worse.

## Prompt shapes for the ladder

| Weak | Strong |
| --- | --- |
| “the build is failing” | “the build fails with this error: [paste]. fix it and verify the build succeeds. **address the root cause, don’t suppress the error**“ |
| “fix the login bug” | “users report login fails after session timeout. check `src/auth/`, especially token refresh. **write a failing test that reproduces it**, then fix” |
| “why is this API so weird?” | “look through `ExecutionFactory`'s git history and summarize how its API came to be” |

The pattern across all three: name the symptom, name the likely location, and name what “fixed” looks like. The third example is the one people forget exists — **git history is context**, and Claude reads it happily.
