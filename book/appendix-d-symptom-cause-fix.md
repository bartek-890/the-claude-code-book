# Symptom → cause → fix

_Appendix D_

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Claude ignores a `CLAUDE.md` rule | File too long; the rule is lost in noise | Prune to under 200 lines; convert enforceable rules to hooks → Ch. 10, 14 |
| A rule works, then stops mid-session | It was in a `paths:` rule or nested file and compaction dropped it | Move it to project-root `CLAUDE.md` → Ch. 03 |
| `CLAUDE.md` doesn’t seem to load | Wrong location, or excluded | `/context` → check **Memory files**; then `claudeMdExcludes` |
| Claude asks about something the file answers | Ambiguous phrasing, not length | Make it concrete and verifiable: “use 2-space indentation”, not “format properly” |
| Agent forgets earlier instructions | Window bloat plus lost-in-the-middle | `/clear`, delegate reads to subagents → Ch. 03 |
| Rules held early, broken by turn 30 | The constraint is still in the transcript but no longer near either edge | Move it into a file that gets re-injected; restate it in the turn that matters → Ch. 03 |
| Mid-feature, it acts like a brand-new agent | Auto-compaction summarised the working context away | `SessionStart` hook with `source: "compact"` re-injects; keep the spec in a file → Ch. 03, 14 |
| A clean diff against code that doesn’t exist | A fabricated path or function became context for later turns | Make it quote the code before changing it; `@`-mention the real file; close with a command that fails loudly → Ch. 03 |
| Session slows down and quality drops | Every turn replays the whole history | `/context`, then `/compact` with instructions |
| A skill never triggers | Description doesn’t name the trigger | Add `when_to_use` with phrases you actually type → Ch. 12 |
| A skill triggers constantly | Description too broad | Narrow it, or `disable-model-invocation: true` |
| Skill A can’t invoke skill B | B has `disable-model-invocation: true` — a hard gate | Remove the flag on B; there is no override |
| Subagent ignores a project rule | It’s `Explore` or `Plan` — they skip `CLAUDE.md` by design | Restate the rule in the delegation prompt → Ch. 13 |
| `/rewind` didn’t restore something | Bash-made change, subagent edit, or a symlinked path | `git revert`. Commit before delegating. → Ch. 26 |
| `/goal` never completes | The proof isn’t in the transcript — the evaluator can’t run tools | Name the command in the condition; make Claude run it each turn → Ch. 22 |
| `/goal` won’t start | Workspace not trusted, or hooks disabled | Accept the trust dialog; check `disableAllHooks` / `allowManagedHooksOnly` |
| Auto mode keeps prompting | Classifier blocked 3 in a row or 20 total | Approve once to resume; configure trusted infrastructure → Ch. 05 |
| A boundary you stated got ignored | Compaction removed the message that stated it | Use a `permissions.deny` rule instead |
| Sandboxed Claude still read `~/.ssh` | Sandbox read access defaults to nearly the whole machine | `sandbox.credentials` **and** `permissions.deny` — both layers |
| Deny rule bypassed by a script | Rules cover recognised file commands, not arbitrary subprocesses | OS-level sandbox → Ch. 05 |
| Edit fails: “old_string not found” | Whitespace mismatch, or the file changed on disk | Re-read the file; ask for a wider unique anchor |
| Grep misses files Glob finds | Grep respects `.gitignore`; Glob does not | Pass the path directly → Ch. 04 |
| WebFetch says a page “doesn’t mention” X | The extraction prompt didn’t ask; Claude never saw the page | Re-fetch with a prompt that asks for X specifically |
| Startup feels heavy | MCP names, skill descriptions, agent roster | `/context`; disconnect unused servers → Ch. 02 |
| Costs rose after you trimmed tokens | The trim broke cache hits worth more than the tokens | Mask or drop turns; never rewrite cached text → Ch. 29 |
| Costs rose and sessions “feel worse” | A broken cache looks exactly like a worse model from outside | Check `cache_read_input_tokens` and what started rewriting your prefix → Ch. 29 |
| A command from a guide says `Unknown command` | Removed, renamed, or gated behind a newer version | `/help` for what your build has; `/release-notes` for when it changed → Ch. 08 |
| Something is broken and you can’t tell why | Your own configuration | `claude --safe-mode`, then `/doctor`, then bisect |
