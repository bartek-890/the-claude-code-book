# Rules, imports, and auto memory

_III · Harness · Chapter 11_

Three mechanisms sit alongside `CLAUDE.md` and are constantly confused with it. **Rules** are instructions you write that can be scoped to file paths. **Imports** are organisational, not economical. **Auto memory** is written by Claude, not by you, and is the only one of the three with a hard size limit.

## Project rules: `.claude/rules/`

Markdown files, one topic each, discovered recursively — so subdirectories like `frontend/` and `backend/` work. A rule **without** `paths:` frontmatter loads at launch with the same priority as `.claude/CLAUDE.md`. A rule **with** it loads only when Claude reads a matching file.

```
---
paths:
  - "src/api/**/*.ts"
  - "src/**/*.{ts,tsx}"
---

# API development rules
- All API endpoints must include input validation
- Use the standard error response format
- Include OpenAPI documentation comments
```

| Pattern | Matches |
| --- | --- |
| `**/*.ts` | All TypeScript files in any directory |
| `src/**/*` | Everything under `src/` |
| `*.md` | Markdown files in the project root only |
| `src/**/*.{ts,tsx}` | Brace expansion — each group multiplies the pattern count |

> **Warning — Three glob gotchas**
>
> - A rule’s whole `paths` list shares a budget of **1,000 expanded patterns** and 4 MiB. Patterns that exceed it are used unexpanded, and literal braces match nothing.
> - `[` starts a bracket expression. `photos [2024/**` is invalid and matches nothing; escape it as `photos \[2024/**`.
> - Path-scoped rules and nested `CLAUDE.md` files are **lost on compaction** until a matching file is read again. If a rule must persist, drop the `paths:` or move it to project-root `CLAUDE.md`. → Ch. 03

User-level rules live in `~/.claude/rules/` and apply to every project on your machine; they load _before_ project rules, giving project rules higher priority. The directory supports symlinks, so a shared company rule set can be linked into many repos:

```
ln -s ~/shared-claude-rules            .claude/rules/shared
ln -s ~/company-standards/security.md  .claude/rules/security.md
```

## Imports: `@path/to/file`

Both relative and absolute paths work; relative resolves against the file containing the import, not the working directory. Imports recurse to a maximum depth of **four hops**. Parsing skips code spans and fenced blocks, so `` `@README` `` in backticks stays literal while `@README` outside them imports the file.

> **Key — The thing everyone gets wrong about imports**
>
> **Imported files are expanded and loaded at launch.** Splitting a 400-line `CLAUDE.md` into four 100-line imports organises your repository and changes your context cost by exactly zero. If you want a size reduction, you need a skill or a path-scoped rule — something that loads _on demand_.

One security note: an import in a _project_ memory file whose path resolves outside the working directory is “external”. The first time Claude Code sees one it shows an approval dialog listing the files; declining disables them permanently and silently. Imports in user-scope files carry the same trust as the rest of your personal configuration and load without a dialog.

## Auto memory

Auto memory is the mirror image of `CLAUDE.md`: Claude writes it, you read it.

|  | CLAUDE.md | Auto memory |
| --- | --- | --- |
| Who writes it | You | Claude |
| What it holds | Instructions and rules | Learnings and patterns it discovered |
| Scope | Project, user, or org | Per repository, shared across worktrees, machine-local |
| Loaded | Every session, in full | Every session, **first 200 lines or 25 KB** |
| Use for | Standards, workflows, architecture | Build commands, debugging insights, preferences it inferred |

Storage is `~/.claude/projects/<project>/memory/`, keyed off the git repository so every worktree shares one directory. `MEMORY.md` is an _index_; topic files beside it are read on demand and never load at startup.

```
~/.claude/projects/<project>/memory/
├── MEMORY.md          # concise index — loaded every session
├── debugging.md       # read on demand
└── api-conventions.md # read on demand
```

> **Note — The index is enforced v2.1.210+**
>
> After Claude writes `MEMORY.md`, Claude Code measures it against the 200-line and 25 KB limits. Near a limit, Claude is reminded to shorten it — one line per entry, detail moved to topic files, stale entries merged or dropped. Over a limit, the write succeeds but Claude Code returns an error telling Claude to rewrite the index, because everything past the limit is dropped on the next load. v2.1.211+ the check ignores frontmatter and HTML comments, which are stripped before loading.

Controls: toggle in `/memory`, or set `autoMemoryEnabled: false` in settings (works per project), or export `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`. Redirect storage with `autoMemoryDirectory` — absolute path or `~/`-prefixed, and honoured from a project scope only after you accept the workspace trust dialog.

When you see “Saved 2 memories” or “Recalled 2 memories” in the interface, that is this system working. Everything it writes is plain markdown you can read, edit, or delete. Audit it occasionally — a wrong memory is worse than no memory, and it persists across every session in the repo.

## Choosing between the four

| Put it in… | …when |
| --- | --- |
| Project `CLAUDE.md` | It applies to every task in the repo and must survive compaction |
| A path-scoped rule | It applies to one part of the codebase and would be noise elsewhere |
| A skill | It is a multi-step procedure, or long reference material |
| A hook | It must happen every time regardless of what the model decides |

Anthropic’s own trigger for adding to `CLAUDE.md` is worth adopting verbatim. Write the rule down when:

- **Claude makes the same mistake a second time.** Once is noise; twice is a pattern the file should catch.
- A code review catches something Claude should have known.
- You type a correction you already typed last session.
- A new teammate would need the same piece of context.
