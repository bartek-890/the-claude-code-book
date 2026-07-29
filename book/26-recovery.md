# Recovery

_IV · Flows · Chapter 26_

Learn these before you need them. The most expensive sessions are not the ones that go wrong — they are the ones where someone kept going for another twenty minutes after they went wrong.

## Four commands, in order of how bad it has got

| Situation | Command | What it does |
| --- | --- | --- |
| Mid-edit and going the wrong way | `Esc` | Interrupts immediately, before the change lands. Context preserved. |
| The last accepted change was wrong | `/rewind` | Reverts to the state before that change |
| The session drifted or context went stale | `/clear` | Wipes context; the next prompt starts fresh |
| Context is long but still relevant | `/compact` | Summarises history instead of discarding it |

## What checkpoints do and do not cover

Every prompt you send creates a checkpoint. Claude Code keeps snapshots for the **100 most recent** in a session, stored with the conversation itself — so a session you resume tomorrow can still rewind to them. They are deleted along with the session after `cleanupPeriodDays` (default 30).

| Tracked | Not tracked |
| --- | --- |
| Edits made through Claude’s file editing tools | **Files modified by Bash commands** — `rm`, `mv`, `cp` |
| Foreground forked-skill edits | **Subagent edits**, including background `/code-review --fix` |
| Files touched in _this_ session | Manual edits you made outside Claude Code, or from another session |
| Regular files | **Symlinked and hard-linked paths** — skipped with a warning v2.1.216+ |

> **Warning — Checkpoints are local undo. Git is history.**
>
> Use `/rewind` for the last accepted edit. Use `git revert` when it is bigger than that — across a multi-file change, git is the only rollback that returns you to an exact, known state with no surprises. If you find yourself wishing checkpoints covered more, the real answer is that you should have committed.

## The rewind menu in full

`/rewind`, or `Esc Esc` on an empty input. Select a prompt, then:

- **RESTORE CODE AND CONVERSATION** — Revert both to that point
- **RESTORE CONVERSATION** — Rewind the thread, keep current code
- **RESTORE CODE** — Revert files, keep the conversation
- **SUMMARIZE FROM HERE** — Compress this message onward; keep earlier detail
- **SUMMARIZE UP TO HERE** — Compress everything before; keep recent detail

The code options appear only when the selected checkpoint has tracked file changes. After restoring the conversation, the original prompt is placed back in the input so you can edit and re-send it. v2.1.191+ If you ran `/clear` earlier in the same process, the menu offers a `/resume <session-id> (previous session)` entry at the top — the escape hatch for clearing something you needed.

## Git habits that make recovery cheap

1. **Start every feature from a clean tree**

   Commit or stash before the agent starts something new. If it goes sideways you discard the working tree and lose nothing you cared about.

2. **Commit after every task that lands green**

   Small commits are the checkpoint system that makes the whole loop safe — the next task can fail freely because the last good state is one command away.

3. **Let the agent do the git chores, keep the git decisions**

   Commit messages, branch creation, push mechanics — all delegable. _What_ to revert and _when_ to commit stay yours.

4. **Require a written migration plan before any schema change ships**

   Forward SQL, a rollback path, a verification step. Schema changes are the one category of edit `git revert` does not cleanly undo once data has landed on the new shape.

## Five failure patterns and their fixes

| Pattern | What it looks like | Fix |
| --- | --- | --- |
| **The kitchen sink session** | One task, then something unrelated, then back to the first | `/clear` between unrelated tasks |
| **Correcting over and over** | Wrong, corrected, still wrong, corrected again | After **two** failed corrections, `/clear` and write a better prompt |
| **The over-specified `CLAUDE.md`** | Claude ignores half the file | Prune ruthlessly; convert enforceable rules to hooks |
| **The trust-then-verify gap** | Plausible implementation, unhandled edge cases | Always provide verification. If you can’t verify it, don’t ship it. |
| **Infinite exploration** | “Investigate X” with no scope; hundreds of files read | Scope narrowly, or delegate to a subagent |

> **Key — The one heuristic to keep**
>
> A clean session with a better prompt almost always outperforms a long session with accumulated corrections. The session that ships is the one that stopped and corrected course early — not the one that never went wrong.
