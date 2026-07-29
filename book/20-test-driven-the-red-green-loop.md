# Test-driven: the red-green loop

_IV · Flows · Chapter 20_

TDD is the cleanest possible closing condition for an agentic loop: the check exists before the code does, it is machine-readable, and “done” is not a judgement call. If you only adopt one flow from this part for non-trivial work, adopt this one.

## The flow

1. **Describe the behaviour, ask for the test, forbid the implementation**

   The last clause matters. Left alone, Claude will helpfully write both, and you lose the red step that proves the test can fail.

   ```
   Write tests for `parseDuration(input: string): number`.
   Cases: "30s" → 30, "5m" → 300, "1h30m" → 5400, "" → throws,
   "abc" → throws, "-5m" → throws.
   Do NOT implement parseDuration yet — tests only.
   ```

2. **Run the tests and confirm they fail for the right reason**

   A test that fails because the module does not exist is fine. A test that fails because the _test_ is wrong is a trap you will not notice later.

   ```
   run the tests and show me the output. confirm each failure is
   "not implemented", not a bug in the test.
   ```

3. **Commit the failing tests**

   Now the red state is a checkpoint you can return to. This step takes four seconds and saves entire sessions.

4. **Implement, with the tests named as the stop condition**

   ```
   Now implement parseDuration until every test passes.
   Do not modify the test file. Run the suite after each change and
   paste the output.
   ```

   **“Do not modify the test file”** is not paranoia. An agent optimising for a green suite will absolutely relax an assertion, and it will be right that this makes the tests pass.

5. **Verify with an independent pass**

   Green tests prove the tests pass, not that the implementation is right — a distinction worth one subagent call.

   ```
   use a subagent to check the implementation isn't overfitting to
   the test cases. report any input that would break it.
   ```

6. **Commit on green, then refactor under the safety net**

## Making it permanent

Two ways to stop relying on remembering:

```
# Option A — the session-scoped version
/goal `npm test tests/duration.test.ts` exits 0 and no test file
      is modified, or stop after 15 turns

# Option B — the permanent version: a Stop hook that blocks on red
{
  "hooks": {
    "Stop": [{
      "hooks": [{ "type": "command",
                   "command": "npm test --silent || exit 2" }]
    }]
  }
}
```

Exit `2` blocks the turn from ending and hands stderr to Claude as the reason. Remember the ceiling: Claude Code overrides after **8 consecutive blocks**.

## Related habits worth the same discipline

- **Ask for tests as each feature is built**, not at the end. End-to-end tests earn the most — they simulate a real user path and catch what users would actually hit.
- **Once tests exist, refactor on a schedule.** Every few sessions, pause feature work and ask for a cleanup pass. Generated code accumulates mess faster than handwritten code; the suite makes cleanup safe.
- **Tell it to keep modules small.** Left alone, an agent piles code into one file. Small files are easier for you to review and easier for the _next session’s_ agent to work with — future context is a resource you are either saving or spending now.
