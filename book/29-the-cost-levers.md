# The cost levers

_V · Economics · Chapter 29_

Cost is `(input × input price) + (output × output price)`. There is no minimum charge, so fewer tokens usually means a smaller bill — **except when the cut lands inside a cached prefix**. Then you pay more for less text. This chapter is mostly about that exception, because it is where careful engineers lose money doing the right-looking thing.

It is worth saying plainly why this part of the book exists at all. Among people running these tools professionally, **the bill is the single most-discussed subject** — roughly a fifth of everything they say to each other about the tooling is about cost, quotas, and limits, and no other topic is close. Not model quality. Not benchmarks. Not new releases. The tools became load-bearing faster than anyone worked out what they cost to depend on, and that gap is what the next few pages are for.

## The asymmetry that decides where to cut

| Side | Relative price | Cacheable | Trim priority |
| --- | --- | --- | --- |
| Stable system / tool prefix | 0.1× after first write | Yes | **Keep stable. Do not rewrite.** |
| Unique per-call input | 1× | No | Trim only if it never enters a cache block |
| Model output | ~5× input | **Never** | **Cut first.** |

Illustrative arithmetic on Sonnet 5 intro pricing: 300 calls a day through a cached 1,500-token system prompt cost about **$0.09/day** in cache reads. The same 300 calls each padding the answer with 300 tokens of prose cost **$0.90/day** in output. The padding costs ten times the “bloat” everyone tries to optimise.

## Prompt caching: the mechanics

A prompt cache is a **prefix match on exact bytes**. Anything that changes above the marker — a timestamp, a request ID, a reordered tool list, a dynamic tool result — invalidates every cached token after it. You still get an answer; you pay full price plus the write premium.

| Token type | Multiplier on base input price |
| --- | --- |
| 5-minute cache write | **1.25×** |
| 1-hour cache write | **2×** — set `"cache_control": {"type":"ephemeral","ttl":"1h"}` |
| Cache read | **0.1×** — a 90% discount on reused input |
| Batch API | **0.5×** — and it **stacks** with caching |

> **Key — The cache-stable prefix rule**
>
> 1. Freeze system prompt + tool schemas **before** the cache marker.
> 2. Put dynamic fields **after** it.
> 3. Never rewrite text that already sits inside a cached block mid-session.
> 4. Verify with `usage.cache_read_input_tokens` on call 2+ of an identical prefix. Zero means the prefix is changing, or shorter than the model minimum — **512 tokens on Opus 5**, 1,024 on Opus 4.8, up to 4,096 elsewhere. Shorter prefixes silently don’t cache, with no error.

If reads stay at zero: diff the serialised request body between call 1 and call 2. Anything that changed above the marker is the bug. Note also that a prefix you call _once_ pays the write premium with no read to offset it — caching a one-shot prompt makes it more expensive.

> **Note — Check the cache before you blame the model**
>
> “It got quietly worse” is the most common complaint about these tools and the hardest one to act on, because from the outside a cache that stopped hitting is indistinguishable from a model that changed: same prompts, sessions that feel worse, a bill that moved without you doing anything. Three checks separate them, and all three are cheap. Look at `cache_read_input_tokens` on repeat calls with an identical prefix. Confirm which TTL your prefix is actually on — a 5-minute window that used to be re-hit inside an hour goes cold between turns while nothing in your code changed. And audit your own harness for what started rewriting the prefix: a new MCP server, a reordered tool list, a plugin reload, a timestamp that crept into the system prompt.
>
> The point is not that the provider is always innocent. It is that **a degradation you can measure is a bug you can fix, and one you can only feel is a story you will retell for a month**. Instrument first. This applies to your own harness with exactly the same force as it applies to anyone else’s infrastructure.

## The finding that should change your instincts

> **Warning — Token reduction is not cost reduction**
>
> A study of **2,848 provider-billed Claude Code runs** (arXiv 2607.12161) tested an arm that removed **38% of raw tool-output tokens**. It paid **6.8% _more_** — 95% confidence interval +2.8% to +11.3%. Prompt-cache traffic was ~87% of cost, so compressing history in place destroyed cache hits worth far more than the tokens saved.
>
> Worse, the compression corrupted the verbatim text the edit tool matched against, dropping successful patches from **27/40 to 15/40**. The trim made the agent both more expensive _and_ less capable.

The lesson is not “never trim”. It is: **trim the output side, and on the input side mask or drop older turns rather than rewriting them in place.** Dropping a turn preserves the byte identity of everything above it; rewriting it does not.

The complementary evidence, on the side that _does_ work: the CROP prompt-optimisation study (arXiv 2604.14214) found that regularising response length cut token consumption **80.6%** with only a nominal accuracy dip on GSM8K, LogiQA, and BIG-Bench Hard.

## Output budgets

Three rules that survive contact with production:

1. **Cap `max_tokens`** to the longest useful answer, not the model default. It is the cheapest control you have.
2. **Demand a schema** — `LABEL | reason`, JSON, an enum — so the model cannot invent essay structure. A schema beats “be concise” every time.
3. **Never compress cached text mid-session.** Mask or drop older turns instead.

## The five levers, ranked by reliability

| Lever | Discount | Caveat |
| --- | --- | --- |
| **Tier routing** | Up to 10:1 | Requires the work to actually be separable. → Ch. 27 |
| **Prompt caching** | 90% on reused input | Byte-fragile; needs a stable prefix |
| **Batch processing** | 50%, stacks with caching | Latency-tolerant work only |
| **Output budgets** | Measured up to 80% of tokens | Watch for truncated answers |
| **Context engineering** | >50% measured on SWE-bench Verified | The one that also improves quality. → Ch. 03 |

Inside Claude Code these are harness decisions, not API parameters. A subagent on Haiku _is_ tier routing. `/clear` between tasks _is_ context engineering. A lean `CLAUDE.md` _is_ a stable prefix. And `/usage` shows where the money actually went, broken down by skill, tool, plugin, and MCP server on paid plans.

> **Note — Measure before and after, on one thing**
>
> Apply an output budget to one noisy endpoint this week, keep the prefix byte-stable, and compare `output_tokens` and `cache_read_input_tokens` before and after. Optimising a whole system at once produces a number you cannot attribute to anything.
