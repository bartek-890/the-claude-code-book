# The ten-file harness

_Appendix C_

The minimal harness that changes agent behaviour mechanically. Every file exists because of a rule in this book. `npx leanharness` installs the set — zero dependencies, no prompts, nothing overwritten without `--force`.

| File | What it changes |
| --- | --- |
| `CLAUDE.md` | 60-line skeleton: four behavioural rules plus placeholders → Ch. 10 |
| `AGENTS.md` | Points every non-Claude tool at `CLAUDE.md` — one rule file, not two drifting copies |
| `docs/start.md` | Idea-to-shipped procedure: plan with the strongest model, build per feature, refactor on a schedule, audit with a fix loop |
| `docs/agent-checklist.md` | Human pre-flight and recovery: scoped prompt, session hygiene, diff review, the debug ladder → Ch. 21 |
| `.claude/skills/verify-done/` | Demands proof before “done”: a named check, its real exit code, an honest report if it fails |
| `.claude/skills/security-audit/` | Pre-ship pass: secrets in code and git history, unprotected entry points, input validation, data exposure |
| `.claude/agents/explorer.md` | Read-only recon on Haiku — verbose reads happen in its window, a ~30-line summary reaches yours → Ch. 13 |
| `.claude/agents/code-reviewer.md` | Fresh-context diff review before commit |
| `.claude/agents/researcher.md` | One self-contained topic per run, researched outside your window |
| `.claude/settings.json` | Denies reads of `~/.ssh/**`, `~/.aws/**`, `.env*` → Ch. 05 |

## Why ten and not thirty

Because the always-loaded part of a harness is the expensive part. `CLAUDE.md`, every skill description, and every agent name load at session start and sit in context for every turn. A starter that installs thirty files is not a stronger harness — it just spends the instruction budget before your first prompt arrives.

The same logic decides what stays out:

- **No `llms.txt`** — it is a spec for websites. Inside a repo it only duplicates `CLAUDE.md`.
- **No session-hygiene skill** — `/clear` and `/compact` are moves only the human can make.
- **No MCP configs, no agent sprawl** — every registered name is startup tax.

## The three that work without discipline

Seven of the ten need someone to act on them. Three change behaviour even when nobody remembers they exist:

1. **`.claude/settings.json`** — there is no built-in credential deny list; by default the agent can read `~/.ssh` and `~/.aws/credentials`. The installed rules close that from the first session.
2. **`.claude/agents/explorer.md`** — its _description_ tells the main agent to delegate verbose investigations proactively, so heavy reads run elsewhere with no checkpoint anyone has to remember.
3. **`.claude/skills/verify-done/SKILL.md`** — triggers when the agent is about to declare a task complete and demands proof in the transcript. “Mostly working” stops being an acceptable final answer.

Source and template contents: github.com/bartek-890/leanharness
