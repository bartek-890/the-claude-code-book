# Model tiers

_V · Economics · Chapter 27_

The choice is an economics problem, not a benchmark one. Most teams default to the top tier for everything because it wins the leaderboard, or stay on the middle tier out of habit — and pay the wrong rate for most steps of an agent workflow either way. The spread runs as high as **10:1** between cheapest and most expensive.

## The stack at a glance

|  | Fable 5 | Opus 5 | Sonnet 5 | Haiku 4.5 |
| --- | --- | --- | --- | --- |
| API ID | `claude-fable-5` | `claude-opus-5` | `claude-sonnet-5` | `claude-haiku-4-5` |
| In / out per M tokens | $10 / $50 | $5 / $25 | $3 / $15 ($2 / $10 intro through 2026-08-31) | $1 / $5 |
| Context window | 1M | 1M | 1M | 200K |
| Max output | 128K | 128K | 128K | 64K |
| Latency | Slower | Moderate | Fast | Fastest |
| Thinking | Adaptive, **always on** | Adaptive, **on by default**; effort `low`–`max` | Adaptive; effort `low`–`max` | No adaptive thinking, no `effort` parameter |
| Knowledge cutoff | Jan 2026 | **May 2026** | Jan 2026 | Feb 2025 |
| Refusal handling needed | **Yes** — `stop_reason: "refusal"` on HTTP 200 | No | No | No |

From Anthropic’s model documentation, current as of 2026-07-27. Opus 4.8, 4.7, 4.6, Sonnet 4.6 and 4.5, and Opus 4.5 remain available as legacy models at the same or higher prices; Opus 4.1 is deprecated and retires 2026-08-05. Verify before you commit a budget — this is the fastest-ageing table in the book.

> **Note — One tier most people will never see**
>
> **Claude Mythos 5** (`claude-mythos-5`) shares Fable 5’s specs and pricing and is offered separately for defensive cybersecurity workflows under Project Glasswing. Invitation-only, no self-serve sign-up. Mentioned here so that a reference to it in someone else’s config is not a mystery.

## Four roles, four tiers

| Role in the workflow | Tier | Why |
| --- | --- | --- |
| **Frontier escalation** — the single hardest step, called directly | Fable 5 | Anthropic’s most capable widely released model, at double Opus’s token rate _plus_ a refusal-handling integration cost no other tier has |
| **Orchestrator** — plans, delegates, holds the long thread | Opus 5 | Frontier intelligence at **half the cost of Fable 5**. A step change over Opus 4.8 in deep reasoning, agentic and long-horizon work, and code review — and the only current model whose knowledge runs to **May 2026**. |
| **Default worker** — most coding and knowledge tasks, solo | Sonnet 5 | Near-Opus agentic quality at a fraction of the cost; same 1M window and 128K output |
| **Parallel worker** — fast, scoped, many at once | Haiku 4.5 | 73.3% on SWE-bench Verified at **4–5× the speed** of Sonnet 4.5, at the cheapest rate. Built to run as a subagent. |

Encoded so scripts and subagent launchers can read it:

```
FRONTIER=claude-fable-5        # rare, hardest-step escalation only
ORCHESTRATOR=claude-opus-5     # long-horizon, high autonomy
DEFAULT=claude-sonnet-5        # daily driver
WORKER=claude-haiku-4-5        # parallel sub-agents
```

> **Note — The ranking daily users give you is not the leaderboard**
>
> Ask people who spend six hours a day inside these tools to rank the tiers and the order that comes back does not track SWE-bench, and often does not track code quality either. What it tracks is **stability, price, rate limits, latency, and trust that the thing will behave tomorrow the way it behaved today**. Neither ranking is wrong; they answer different questions. Benchmarks measure capability in a controlled run. Practitioners are reporting the cost of depending on something all day.
>
> The practical use of that: when a colleague says a model “is better”, ask which of the five they mean. Half the model-choice arguments on a team are two people ranking on different axes, and the answer is usually a routing rule rather than a winner.

## The decision rules

1. **Default to Sonnet 5 and escalate _from_ it, with a reason**

   Run it at **medium effort or lower** for the default role. Its cost-performance curve overlaps Opus’s range at every effort level, but the advantage narrows or reverses at high and xhigh — so escalate to Opus, not to Sonnet at xhigh, when you need frontier accuracy.

2. **Reach for Opus 5 when the task is long-horizon and expensive to get wrong**

   A multi-day refactor, a codebase-scale migration, an orchestrator keeping a plan straight across hundreds of tool calls. Two operational notes: **give it the whole spec up front and run at high effort** — a complete first turn produces more efficient _and_ more accurate runs than drip-feeding. And fast mode is an Opus-tier lever nothing else has.

   Opus 5 converts additional effort into better results more reliably than any earlier Opus, so the level you pick carries more weight. Start at the default (`high`) and move in either direction on the evidence of your own evals — down where quality holds, up for the most demanding work. At `xhigh` or `max`, set a large `max_tokens` so it has room to think _and_ act across subagents and tool calls.

3. **Reserve Fable 5 for one step, not one session**

   Every call needs to branch on `stop_reason: "refusal"`, since its safety classifiers can decline a request as a successful HTTP 200 rather than an error. Most teams wire a fallback to Opus for the retry. At double Opus’s rate plus that integration surface, it is an escalation tier, not a default.

4. **Fan Haiku out, and keep each job small**

   The one constraint to design around: **200K context, not 1M.** Give each worker a small self-contained job rather than a sprawling history. It also takes no `effort` parameter — that returns an error.

## Opus 5 changes what you should write in your prompts

Most model upgrades are transparent — same prompts, better answers. Opus 5 is not, and the differences are behavioural rather than API-level. Four you will notice without changing a line of code:

| What changed | What to do about it |
| --- | --- |
| **It verifies its own work unprompted** | **Remove verification instructions carried over from earlier models.** “Include a final verification step”, “use a subagent to verify” — these now cause _over_-verification. |
| It delegates to subagents more readily | Expect more parallel work by default. Multi-agent runs show effective writer–verifier behaviour and few cases of agents overwriting each other. |
| It narrates progress more often | Fine when you are watching; noise in a piped `-p` run. Ask for terse output where that matters. |
| Responses and deliverables run longer | An output budget matters more than it used to. → Ch. 29 |

> **Warning — The one that trips upgraders**
>
> **Thinking is on by default** on Opus 5, where Opus 4.8 ran without it unless asked. Since `max_tokens` is a hard limit on _total_ output — thinking plus response text — revisit it for any workload that previously ran thinking-free. And `thinking: {"type": "disabled"}` is now accepted **only at effort `high` or below**; combining it with `xhigh` or `max` returns a 400. That is a breaking change from Opus 4.8, enforced per request.

Two smaller wins are worth knowing. First, the **minimum cacheable prompt on Opus 5 is 512 tokens**, down from 1,024 on Opus 4.8. Prompts that used to be silently too short to cache now cache, with no change to your code (→ Ch. 29). Second, **mid-conversation tool changes** (beta, `mid-conversation-tool-changes-2026-07-01`) let you add or remove tools between turns _without losing the prompt cache_. One tool list no longer has to be frozen for the life of a session.

## The orchestrator–worker pattern

The tiers compose. An Opus orchestrator delegates scoped sub-tasks to Haiku workers, with Sonnet as the solo default when no orchestration is needed and Fable called directly for the rare step that needs it.

|  |  |
| --- | --- |
| **Symptom** | A single big model reads a 30K-token API reference or a pile of source files, and that clutter stays in context for the rest of the run, dragging quality down. |
| **Cause** | Heavy reads done in the main loop persist on every subsequent turn. |
| **Solution** | Push the read into a subagent that runs in its own window and returns only the conclusion — spawned on Haiku, so you get the isolation _and_ the cheapest token rate for the grunt work. |

> **Key — Why the per-token spread matters more than the benchmark spread**
>
> In a fan-out architecture the **worker tier runs far more tokens than the orchestrator**. A cheaper worker is therefore a cheaper bill on the dominant cost line — and that spread runs up to 10:1 across the full stack, not just within the three workhorse tiers. Tier selection is the only lever that changes your unit price without changing anything you do.

In Claude Code, the practical expression of all of this is one line of subagent frontmatter: `model: haiku`. Define an `Explore` override with it and every codebase survey in every session runs at the cheapest tier that handles the job. → Ch. 13
