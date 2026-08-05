# Sources

_Appendix F_

## The three headline numbers

Three figures do more work in this book than any others, and they are the ones worth checking first. Each is given here with its origin, what it actually measured, and how far it travels.

**~84% fewer permission prompts from sandboxing** — Anthropic’s own figure for Claude Code’s OS-level Bash sandbox: _“The result was an 84% reduction in permission prompts.”_ The same write-up gives the reason the prompts were worth replacing: telemetry showed users approving roughly **93%** of them, which is approval fatigue, not review. This is the firmest of the three — Anthropic measuring its own product. anthropic.com/engineering/how-we-contain-claude

**85% cut in upfront tool tokens from deferred loading** — measured on the Claude API’s tool search tool, which cut tool definitions from **~77K to ~8.7K tokens** before any work begins, preserving 95% of the context window. **Read the scope carefully:** this was measured on the API, not on Claude Code’s MCP loading. Chapter 2 applies it by analogy because the mechanism is the same one — definitions kept out of the prefix until needed — but the number itself is not a Claude Code measurement. anthropic.com/engineering/advanced-tool-use

**150–200 instruction adherence ceiling** — the softest of the three, and the one to treat as a rule of thumb rather than a measurement. Anthropic publishes no such figure; it is a practitioner estimate, consistent with the `CLAUDE.md` length reports below, and it matches what I see in my own harnesses. Use it as an order of magnitude — the argument in Chapters 2 and 10 is that the budget is _finite_, which holds whether the true number is 120 or 250.

## Anthropic documentation

**Claude Code documentation** — hooks reference, commands, interactive mode, CLI reference, memory, settings, status line, permissions and permission modes, sandboxing, skills, subagents, checkpointing, MCP, channels, plugins and plugin marketplaces, best practices. code.claude.com/docs

**Claude Platform documentation** — model overview and pricing, prompt caching, batch processing. platform.claude.com/docs

**Anthropic Engineering** — _Equipping agents for the real world with Agent Skills_; _Effective context engineering for AI agents_. anthropic.com/engineering

## Research and measured findings

**Token Reduction Is Not Cost Reduction** — 2,848 provider-billed Claude Code runs; a 38% tool-output token cut paid 6.8% more (CI +2.8% to +11.3%); patch success fell 27/40 → 15/40. arXiv 2607.12161

**CROP** — response-length regularisation cut token consumption 80.6% with a nominal accuracy dip on GSM8K, LogiQA, BIG-Bench Hard. arXiv 2604.14214

**PwC prompt-caching field study** — 41–80% measured savings from caching. arXiv 2601.06007

**JetBrains Research** — context management cut coding-agent cost by over 50% on SWE-bench Verified.

**Lost in the middle** — the mid-context recall failure, replicated across long-context models.

**METR, 2025** — 16 experienced maintainers, 246 real issues in their own mature repositories: **19% slower** with AI while estimating they had been 20% faster, an average error METR later put at **40 percentage points**. Its February 2026 follow-up was redesigned mid-flight after 30–50% of participants declined to submit tasks they did not want to do without AI, and its authors call the central estimate “likely a bad proxy”. Cited in Chapter 22 for the measured-versus-believed gap, which has outlived every speed number around it — not as a claim that agents are slow.

**Replit production-database deletion, July 2025** — an agent deleted a live database during an explicit code freeze, then reported the rollback as impossible; the restore in fact worked. The fix Replit shipped was automatic dev/production database separation, not a stronger prompt. → Ch. 22

**claude-code issues #37314 and #37847** — user reports that Claude saves a rule to memory and then violates it. Both were closed as stale by a bot with no maintainer reply, which is why Chapter 11 measures the claim rather than repeating it.

**Dreams research preview** — Anthropic’s memory-consolidation pass for Managed Agents, built because memory writes “are local and incremental: over many sessions a memory store accumulates duplicates, contradictions, and stale entries.” Not available in Claude Code. platform.claude.com/docs

**Armin Ronacher** — reverse-engineering of Claude Code’s plan mode: a short injected prompt plus a markdown plan file, not a tool lockdown.

**HumanLayer, Builder.io** — practitioner reports on `CLAUDE.md` length, the ~60-line target, and the ~300-line discount ceiling.

## Field data

Three findings in this book come from working practice rather than a paper, and are given here with what they actually measured. Each describes one engineering context, not the industry — treat them as well-shaped evidence for a mechanism, not as population statistics.

**Adversarial review outcomes** — 83 completed tickets put through an independent second-model review; 67 ended in a real change to the code, and the 122 commits those reviews drove sorted into correctness/logic **34%**, concurrency/races **22%**, durability/data **19%**, security/injection **9%**. One team’s tickets and one team’s definition of each category; the shape — that the finds cluster in defects invisible to a diff skim — is the part that travels. → Ch. 23

**Practitioner discussion, classified** — 2,611 high-engagement discussions among daily users over twelve months to July 2026, roughly 2,046 distinct authors, classified by topic and opinion and weighted by engagement. Cost, quotas, and usage limits accounted for about **22%** — the largest single topic, with nothing close behind. The sample deliberately over-represents the most visible discussion, so it describes the dominant conversation among heavy users, not the average user’s experience. → Ch. 27, 29

**Advisory versus enforced rules** — the same rule stated in `CLAUDE.md` versus implemented as a blocking `PreToolUse` hook, run repeatedly on the same task: routine violation in the first case, none in the second. A single rule in a single repo, which is enough to establish the category difference and not enough to put a number on it. → Ch. 14

**Which layer a rule survives in** — one rule, five layers, two prompts, on Claude Sonnet 5 and Claude Code 2.1.220, 2026-07-30. n=3 per cell, $5.71 across 30 recorded CLI calls, pass/fail decided by a before/after SHA-256 rather than by reading prose. Where the user’s request only tempted the violation, auto memory and a byte-identical `CLAUDE.md` both held 3/3; where the prompt named the forbidden action, both fell to 0/3 and only a `Stop` hook kept the agent from claiming “done” it had not earned. n=3 separates 0/3 from 3/3 and resolves nothing finer, the memory was planted rather than accumulated, and rule wording is a live variable alongside layer. → Ch. 11

**A deny rule observed firing** — the same rig, ordering `rm -rf logs/` with a plausible reason and a pre-empted confirmation. No harness and a bloated `CLAUDE.md` both lost the directory; `"deny": ["Bash(rm -rf *)"]` produced two blocked calls and an intact directory. A companion case in the same run — paste a planted `.env` secret — was refused by every variant including the ones carrying no rule, so it measured nothing about the rule and is reported here as the counter-example it is. → Ch. 05

## Companion articles on bartlomiejkrupa.dev

**Mechanism** — How Claude Code’s agentic loop fills your context window · Context engineering beats a bigger context window · Prune the log, not the window · Claude Code tools · Claude Code security in 2026

**Harness** — Why agents ignore your CLAUDE.md · Claude follows its own memory until you contradict it · How to set up a minimal Claude Code harness · Claude Code skill composition · Subagent context isolation · Claude Code channels

**Flows** — Claude Code plan mode · Claude Code `/goal` · One-way doors your agent can already walk through · The vibe-coding field manual · How to plan and scope a build with an LLM · Verifiable completion condition · The `/goal` evaluator only reads the transcript

**Economics** — Why most agents default to the wrong Claude tier · Why fast mode isn’t a lower effort level · Stop paying full price for repeat LLM calls · Don’t break the prompt cache · Trim output, not the cache · Cache-stable prefix

**Evals** — Five metrics for an agent eval pipeline · LLM-as-a-judge · Verifiable completion condition

**Reference** — Claude Code commands cheatsheet (2026) · Vibe coding vs. agentic engineering · From symbolic AI to the agentic era

> **Key — A closing note on maintenance**
>
> Claude Code ships multiple releases a week. The mechanisms in Parts I and III are stable — the loop, the context window, the split between advisory and deterministic layers have not changed shape in a year and are unlikely to. The _numbers_ are not: prices, tool counts, command lists, and version gates all move. When this book and your terminal disagree, run `/help`, `/context`, and `claude --help`, and trust those.
>
> **The one thing worth carrying out of all of it:** give the agent a check it can run, and review the evidence rather than the claim. Every other practice in this book is a way of making that cheaper.

The Claude Code Book · Edition 2026.08 · Bartłomiej Krupa · bartlomiejkrupa.dev
