# Glossary

_Appendix E_

- **Agentic harness** — The runtime around a language model providing tools, context management, and an execution environment. The model plans; the harness acts.
- **Agentic loop** — Gather → act → verify, cycled until the task is done.
- **Auto memory** — Notes Claude writes for itself across sessions, indexed by `MEMORY.md` (first 200 lines or 25 KB load at startup).
- **Cache-stable prefix** — The byte-identical front of a request later calls reuse at ~0.1× input price. Anything dynamic must sit after the cache marker.
- **Channel** — An MCP server that pushes external events into a running local session.
- **Checkpoint** — A snapshot of tool-made file edits taken before each user prompt; restorable with `/rewind`. Not a replacement for git.
- **Completion condition** — The plain-text end state a `/goal` run works toward, judged by a separate evaluator model.
- **Context engineering** — Curating what enters the context window each turn rather than relying on a bigger window.
- **Effort** — A parameter scaling all response tokens — text, tool calls, and thinking together.
- **Evaluator** — The small fast model that judges a `/goal` condition after every turn. It does not call tools.
- **Fast mode** — An Opus-tier setting buying up to 2.5× speed at a flat per-token premium. Identical output quality.
- **Fork** — A subagent that inherits the parent conversation and system prompt — the exception to subagent isolation.
- **Hook** — Code that runs at a fixed lifecycle event regardless of what the model decides. The only deterministic layer in the harness.
- **Lost in the middle** — The observed failure to reliably use information placed mid-context; recall is strongest at the start and end.
- **MCP** — Model Context Protocol — the open standard for connecting external tools and data to an agent.
- **One-way door** — A decision that is irreversible or nearly so; a two-way door is one you can reopen and walk back through. Reversal cost is knowable before you prompt, unlike speed, which makes it the axis worth routing a delegation on.
- **Permission mode** — The session setting governing which actions run without asking: `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions`.
- **Plan mode** — A permission mode where Claude reads and plans but makes no edits until you approve. Prompt-enforced, not sandboxed.
- **Progressive disclosure** — Keeping the always-loaded surface small and pushing detail into files that load on demand.
- **Skill** — A markdown file whose body loads only when used, packaging a workflow or reference material.
- **Startup tax** — Tokens loaded into the window before your first message, paid on every new session.
- **Subagent** — A delegated agent running in its own context window, returning only a summary.
- **Verifiable completion condition** — A stop rule with one measurable end state the model can prove from its own output — the difference between a loop that terminates and one that spins.
