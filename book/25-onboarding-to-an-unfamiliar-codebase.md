# Onboarding to an unfamiliar codebase

_IV · Flows · Chapter 25_

The least-used and highest-return application of Claude Code: not writing code, but _reading_ it on your behalf. Ask the questions you would ask a senior engineer, and you get the ramp-up without the interruption cost.

## Questions that work verbatim

- How does logging work here?
- How do I make a new API endpoint? Show me the smallest existing example.
- What does `async move { … }` do on line 134 of `foo.rs`?
- What edge cases does `CustomerOnboardingFlowImpl` handle?
- Why does this code call `foo()` instead of `bar()` on line 333?
- Look through `ExecutionFactory`'s git history and summarise how its API came to be.
- What would you improve in this file?

The last one is a deliberate exception to the “be specific” rule. A vague prompt is useful precisely when you are exploring and can afford to course-correct — it surfaces things you would not have thought to ask about.

## The flow

1. **Delegate the survey, don’t run it in your window**

   ```
   Use subagents to map this repo: the entry points, the request
   lifecycle, where state lives, and what the test strategy is.
   Return a one-page summary with file paths. Don't paste file contents.
   ```

   This is the difference between a 5K-token answer and a 90K-token session that is useless by the time you ask your second question.

2. **Ask your way down from the summary**

   Follow the paths the summary named. Each question is now specific, which keeps the reads small.

3. **Write what you learned into `CLAUDE.md` — but only the non-obvious parts**

   Run `/init` if there is no file yet: it detects build systems, test frameworks, and code patterns and gives you a foundation. Then apply the one-minute test to everything it generated. → Ch. 10

4. **`/clear` before you write your first line of code**

   The exploration was research. Carrying it into implementation is how a clean window becomes a cluttered one before the first edit.

## Give it the right tools for the job

- **CLI tools are the most context-efficient integration available.** Install `gh` and Claude uses it for issues, PRs, and comments. Without it, unauthenticated API requests hit rate limits. The same goes for `aws`, `gcloud`, `sentry-cli`.
- **Claude learns unfamiliar CLIs on request:** “use `foo-cli --help` to learn the tool, then use it to solve A, B, C.”
- **A code-intelligence plugin** gives precise symbol navigation and automatic error detection after edits in a typed language — far better than grep for “who calls this?”.

> **Key — Onboarding is a load-bearing use case**
>
> Used this way, Claude Code reduces ramp-up time _and_ the load on the engineers who would otherwise answer these questions. No special prompting required — ask directly. It is also the safest possible first week with the tool: read-only, no permission decisions, and every answer is verifiable against the file it names.
