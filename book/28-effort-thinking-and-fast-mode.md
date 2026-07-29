# Effort, thinking, and fast mode

_V · Economics · Chapter 28_

Three separate controls, constantly mistaken for one setting. **Effort** scales every token Claude spends. **Thinking budget** caps reasoning tokens specifically, and only on older fixed-budget models. **Fast mode** changes neither — it pays a flat premium for the same model to answer up to 2.5× faster. Anthropic’s own documentation recommends stacking them, not picking one.

## The levers at a glance

| Lever | What it scales | Cost effect | How to set it |
| --- | --- | --- | --- |
| Effort level | All response tokens: text, tool calls, thinking | Lower level = fewer tokens = lower cost | `/effort`, `--effort`, `CLAUDE_CODE_EFFORT_LEVEL`, `effortLevel` |
| Thinking budget | Reasoning tokens only, fixed-budget models only | Thinking bills as output tokens | `MAX_THINKING_TOKENS`, `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` |
| Fast mode | **Nothing** — same tokens, same quality | Flat premium per token | `/fast`, `fastMode`, `Option+O` |

## Effort: the token-spend dial

Effort is a _behavioural signal, not a hard cap_. At low effort Claude still thinks on a genuinely hard step — just less than it would at high effort on the same step. It affects every token in the response, which is what makes it different from a thinking-only control: lower effort also means **fewer tool calls and terser explanations**, not just less reasoning.

| Level | What it trades | Typical use |
| --- | --- | --- |
| `max` | No constraint on token spend | Frontier problems, deepest reasoning |
| `xhigh` | Extended capability for long-horizon work | Agentic tasks over 30 minutes, million-token budgets |
| `high` (default on most models) | Balanced depth and cost | Complex reasoning, difficult coding |
| `medium` | Moderate savings | Agentic work trading some depth for speed |
| `low` | Biggest savings, some capability loss | **Subagents** — Anthropic names them explicitly |

Availability varies: Fable 5, Opus 5, Sonnet 5, and Opus 4.8 support all five levels; Opus 4.6 and Sonnet 4.6 skip `xhigh`; **Haiku 4.5 has no `effort` parameter at all** and returns an error if you send one. The default is `high` on Opus 5 and Sonnet 5 in Claude Code and on the Claude API.

> **Note — Precedence, and two names that aren’t levels**
>
> `CLAUDE_CODE_EFFORT_LEVEL` wins over everything, including a skill’s or subagent’s `effort:` frontmatter. Below it: frontmatter → `--effort` → `/effort` → `effortLevel` in settings.
>
> **`ultracode`** is not a sixth level — it sends `xhigh` to the model and lets Claude Code orchestrate multiagent workflows, session-only. **`ultrathink`** dropped anywhere in a prompt requests deeper reasoning for that one turn without touching the session setting at all.

## Extended thinking: adaptive versus fixed budget

Fable 5, Opus 5, Sonnet 5, and Opus 4.7+ always use **adaptive reasoning** — the model decides per step whether to think and how much, guided by effort. On these, a nonzero `MAX_THINKING_TOKENS` is ignored; effort is the only real dial. Opus 4.6 and Sonnet 4.6 default to adaptive too but can revert with `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1`, at which point `MAX_THINKING_TOKENS` sets an actual ceiling.

```
# session-wide spend dial — on adaptive models, the only thinking control
export CLAUDE_CODE_EFFORT_LEVEL=medium

# fixed-budget models only: revert from adaptive, then set a ceiling
export CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1
export MAX_THINKING_TOKENS=8000
```

`MAX_THINKING_TOKENS=0` turns thinking off entirely on the Anthropic API, on any model, with two exceptions. **Fable 5 cannot disable thinking under any setting** — the session toggle, `alwaysThinkingEnabled`, and `MAX_THINKING_TOKENS=0` all have no effect there. And on **Opus 5, disabling thinking is only accepted at effort `high` or below**; at `xhigh` or `max` the request returns a 400. Anthropic’s own guidance is to leave thinking on and control cost with a lower effort level instead. There is a correctness reason as well as a cost one: with thinking disabled, Opus 5 occasionally writes a tool call into its text output instead of emitting a proper tool-use block.

Thinking tokens **bill as output tokens whether or not you ever look at them** — collapsed by default, expandable with `Ctrl+O`. That is the direct link to cost: a model at `high` effort with adaptive reasoning can spend tens of thousands of thinking tokens before writing a single word of visible output.

## Fast mode: paying for latency

A research-preview Claude Code setting, Opus-only, that runs the same model through a different API configuration prioritising speed. **Not a cheaper or smaller model** — output quality is identical; only arrival speed and per-token price change.

| Model | Fast-mode in / out per MTok | Notes |
| --- | --- | --- |
| **Opus 5** | $10 / $50 | Up to 2.5×. The fast-mode default in v2.1.219+ |
| Opus 4.8 | $10 / $50 | Up to 2.5×. Was the default in v2.1.154–2.1.218 |
| Sonnet · Haiku · Fable | — | **Not supported.** Enabling fast mode switches you to Opus. |

Pricing is flat across the full 1M-token window. Toggle with `/fast` (then `Tab`), `Option+O`, or `"fastMode": true`. It is **not supported in the VS Code extension.** Turning it off leaves you on Opus — the model does not revert; use `/model` for that.

> **Warning — The Opus 4.7 trap**
>
> Fast mode for Opus 4.7 was deprecated 2026-06-25 and **removed 2026-07-24**. Claude Code has not caught up: it still treats 4.7 as a fast-mode model everywhere it decides whether fast mode is on — the toggle, model switches, session start. **The API then rejects those requests rather than serving them at standard speed.** If a session on 4.7 starts failing after a `/fast`, that is why. Switch to Opus 5.

Two operational details that surprise people. Fast mode **draws from usage credits even when you have plan usage remaining** — those tokens do not count against your included usage and bill at the fast rate from the first token. And it has a **separate rate-limit pool shared by every supported Opus model**; when you hit it, fast mode falls back to standard speed on its own, the `↯` icon greys out, and it re-enables after the cooldown. Nothing breaks; you just quietly stop paying the premium.

> **Warning — Turn it on at the start of a session, not the middle**
>
> The **first fast-mode message in a conversation pays the full, uncached fast-mode input price for the entire existing context** — not just the new message. That cost applies once per conversation; toggling off and back on later does not repeat it. But switching on midway through a long session means paying the premium on everything you have already accumulated.
>
> Fast mode also carries over into future sessions by default. Set `fastModePerSessionOptIn: true` to make every new session start with it off.

## How to combine them

- **SIMPLE TASK, WANT IT NOW** — Low effort + fast mode — fewer tokens, each arriving faster
- **QUALITY MATTERS MOST** — Drop fast mode, raise effort
- **SUBAGENT WORK** — Low effort, cheap tier, no fast mode
- **LONG-HORIZON MIGRATION** — Opus at xhigh, full spec up front, no fast mode

The reasoning behind the second row is worth stating plainly: **paying more per token for identical output makes no sense next to a setting that changes what the model actually produces.** Fast mode buys wall-clock time and nothing else. Spend on it only when your own time is the constraint.
