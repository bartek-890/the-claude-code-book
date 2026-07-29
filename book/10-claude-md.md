# CLAUDE.md

_III · Harness · Chapter 10_

If your agent skips instructions you put in `CLAUDE.md`, the file is almost certainly **too long and too specific**. The fix is not louder emphasis. It is a shorter, more universal file, with everything else moved behind progressive disclosure — loaded only when a task calls for it, not on every turn.

- **CLAUDE.md** — A project file Claude Code loads at the start of every session as standing instructions. It is the agent’s onboarding surface — agents start each session with no memory of your codebase.
- **Progressive disclosure** — The pattern of keeping the always-loaded surface small and pushing detail into files that load only when relevant: skills, path-scoped rules, imports, reference docs.

## Two failure modes, both worsened by length

### Relevance filtering

When much of the file reads as irrelevant to the task at hand, the agent discounts the whole thing — _including the parts that mattered_. Practitioners writing about Claude Code (HumanLayer, Builder.io) consistently report that non-universal, task-specific content raises the ignore rate.

### Instruction budget

Every instruction dilutes the others. Reliable adherence sits at roughly **150–200 instructions** for frontier thinking models — a practitioner rule of thumb rather than a published measurement (Appendix F) — and Claude Code’s system prompt already spends about **50** before your file loads. Padding `CLAUDE.md` with situational rules burns a finite attention budget on low-value lines.

The through-line: **task-specific content in a file that loads for every task is the problem.**

## What goes in, what stays out

| Include | Exclude |
| --- | --- |
| Non-guessable shell commands | Anything obvious from the code or types |
| The project’s test runner | Restating existing docs |
| Code style that differs from language defaults | One-off, task-specific steps |
| Repo etiquette: branch naming, PR conventions | Long examples better kept in a linked doc |
| Architecture decisions a newcomer can’t infer | Style rules a formatter already enforces |
| Environment quirks and required env vars | Information that changes frequently |
| Common gotchas and non-obvious behaviours | File-by-file descriptions of the codebase |

> **Key — The one-minute test**
>
> If a competent new contributor could figure it out from the repo in a minute, it does not belong in `CLAUDE.md`. Apply it to every line, every time you add one.

Anthropic’s own phrasing of the same test, per line: _“Would removing this cause Claude to make mistakes?”_ If not, cut it.

## How long is too long

Anthropic’s documentation targets **under 200 lines** per file. HumanLayer keeps theirs **under 60**. Around **300 lines** is the ceiling practitioners cite before agents visibly discount the file. Note the file is loaded _in full_ regardless of length — there is no truncation to save you.

Two diagnostic symptoms worth learning:

- **Claude keeps doing something you have a rule against.** The file is probably too long and the rule is getting lost.
- **Claude asks a question answered in the file.** The phrasing is probably ambiguous, not the length.

## A lean file that survives the relevance filter

```
# CLAUDE.md

## Commands
- Test:  pnpm test            # not npm — pnpm workspaces
- Build: pnpm build
- Check: pnpm check           # types + content schemas

## Architecture
- Static Astro site; content lives in src/content/ as MDX
- The `pl` collection is isolated from English feeds — do not cross-link

## Conventions
- Formatting is Prettier + ESLint. Run the tools, don't restate the rules.

## Compact Instructions
When compacting, preserve: current task goal, files being edited,
test commands, open decisions.

@docs/architecture.md   # pulled in only where referenced
```

Four sections and one import. Everything a formatter enforces, the framework documents, or the type system expresses is absent by design.

## Where the rest goes

- **@path IMPORTS** — Organise — but they still load at launch. Not a size fix.
- **SKILLS** — A domain workflow loaded when the task calls for it → Ch. 12
- **PATH-SCOPED RULES** — Instructions that load when a matching file is read → Ch. 11
- **CHILD-DIRECTORY CLAUDE.md** — Scoped to a subtree, loaded on demand
- **REFERENCE DOCS** — The big architecture map or PRD, out of the always-on file
- **HOOKS** — Anything that must happen every time, no exceptions → Ch. 14

> **Warning — Don’t use the model as a linter**
>
> Style and formatting rules are cheaper and far more reliably enforced by tooling — a formatter, a Stop hook, a slash command — than by instructions the agent must remember every turn. Reserve `CLAUDE.md` for what only prose can convey.

## Where files live, and in what order

| Scope | Location | Shared with |
| --- | --- | --- |
| Managed policy | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`<br>Linux/WSL `/etc/claude-code/CLAUDE.md`<br>Windows `C:\Program Files\ClaudeCode\CLAUDE.md` | Everyone in the org. **Cannot be excluded.** |
| User | `~/.claude/CLAUDE.md` | You, all projects |
| Project | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team, via version control |
| Local | `./CLAUDE.local.md` | You, this project. Gitignore it. |

Claude Code walks _up_ from your working directory, loading `CLAUDE.md` and `CLAUDE.local.md` at every level. Files are **concatenated, not overridden**, ordered filesystem-root-down — so instructions closer to where you launched are read last, and `CLAUDE.local.md` is appended after `CLAUDE.md` at each level. Files in _sub_directories load on demand, when Claude reads a file there.

> **Note — Two practical details**
>
> - **Block-level HTML comments are stripped** before injection. Use `<!-- maintainer note -->` for humans without spending context tokens.
> - **`CLAUDE.md` is delivered as a user message after the system prompt**, not as part of it. That is why compliance is strong but not guaranteed, and why `--append-system-prompt` exists for instructions that must sit higher.

## AGENTS.md and the other tools

Claude Code reads `CLAUDE.md`, not `AGENTS.md`. If your repo already uses `AGENTS.md` for Codex, Cursor, or others, do not maintain two drifting copies — import it:

```
<!-- CLAUDE.md -->
@AGENTS.md

## Claude Code
Use plan mode for changes under `src/billing/`.
```

A symlink works too (`ln -s AGENTS.md CLAUDE.md`) when you have nothing Claude-specific to add. On Windows, symlinks need Administrator or Developer Mode — use the import.

`/init` reads Cursor rules (`.cursor/rules/`, `.cursorrules`) and Copilot instructions (`.github/copilot-instructions.md`) and folds the relevant parts in. With `CLAUDE_CODE_NEW_INIT=1` it also reads `AGENTS.md`, `.devin/rules/`, `.windsurf/rules/`, and `.clinerules`, runs an interactive multi-phase flow, and presents a reviewable proposal before writing anything.

## Migrating a file that has already gone wrong

1. **Move the old file aside**

   `mv CLAUDE.md CLAUDE.old.md`. Do not try to edit in place — you will preserve lines out of sunk-cost reflex.

2. **Start from a skeleton**

   Either the shape above, Appendix B, or `npx leanharness`, which installs a 60-line skeleton plus the checklist, skills, and subagents that keep it honest.

3. **Port only what survives the one-minute test**

   Non-guessable commands, the test runner, architecture decisions, real conventions. Straight into the placeholders.

4. **Delete the old file once nothing left in it passes**

   The usual casualties: lint rules the linter config enforces, framework walkthroughs the framework docs own, style guides the formatter applies.

5. **Verify it loaded**

   Run `/context` and check the **Memory files** list. If it is not there, Claude cannot see it, and no amount of rewriting will help.

Two cases where you should not bother: a harness that is already tuned (you will get a list of skipped files and nothing else), and a monorepo with per-package rule files (nested files stay your job).
