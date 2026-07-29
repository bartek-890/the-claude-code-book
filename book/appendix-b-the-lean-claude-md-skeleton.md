# The lean CLAUDE.md skeleton

_Appendix B_

Under 60 lines when filled. Delete every comment before committing — they are guidance for you, not context for the agent. Anything you are tempted to add: apply the one-minute test first.

```
# CLAUDE.md

## Behaviour
1. Think before coding. State assumptions; ask when uncertain; present
   alternatives instead of picking silently; push back when a simpler
   approach exists.
2. Simplicity first. Minimum code that solves the problem. No unrequested
   features, abstractions, configurability, or error handling for
   impossible cases.
3. Surgical changes. Touch only what the request requires. Match existing
   style. Remove only imports and helpers your change orphaned; mention
   pre-existing dead code, don't delete it.
4. Goal-driven execution. Turn tasks into verifiable goals. State brief
   plan steps with verification checks for multi-step work.

## Commands
# Only what can't be guessed from package.json / Makefile / README.
- Test:  <non-default test runner>
- Build: <non-guessable build command>
- Check: <typecheck / schema validation>

## Architecture
# One line per decision a newcomer could not infer in a minute.
- <decision>
- <boundary that must not be crossed>

## Conventions
# Only what a formatter or linter cannot enforce.
- <convention>

## Compact Instructions
When compacting, preserve: current task goal, files being edited, test
commands, and open decisions. Drop: exploratory reads, superseded plans,
verbose tool output.

## Progressive disclosure
@docs/architecture.md   # big maps live here, not above
```

> **Note — Why the behaviour block sits at the top**
>
> Those four rules are the only genuinely universal content in the file — they apply to every task in every repo, which is exactly the test for belonging in an always-loaded surface. Everything below them is project-specific by construction, and everything project-specific should be short.
