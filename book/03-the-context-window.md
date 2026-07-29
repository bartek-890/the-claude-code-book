# The context window

_I · Mechanism · Chapter 03_

Flagship models in Claude Code run up to a **1M-token** window. A bigger window does not fix a cluttered one. Two mechanics explain why nearly every “the agent forgot” complaint is a context problem, not a memory problem.

- **Context window** — The maximum number of tokens the model can consider at once: system prompt, full conversation history, tool results, and its own replies.
- **Context engineering** — Deliberately curating what enters that window each turn, instead of trusting a larger window to absorb the clutter.

## Mechanic one: every turn replays everything

An agent turn is not just your latest prompt. The model re-reads the entire state each time — prior messages, its own previous responses, tool outputs, file reads, fetched docs. A single large file read keeps costing tokens for the rest of the session. This is why “just read the whole directory” is expensive in a way that feels free at the moment you ask for it.

## Mechanic two: the middle gets lost

Recall is strongest at the very start and very end of the window and weakest in between — the documented **“lost in the middle”** effect. A fact buried mid-context can be ignored while the token count is still far under the cap. **Quality drops before the hard limit, not at it.** Two consequences for how you write prompts:

- Put the most important instructions and data near the **top or bottom** of a long prompt.
- Trim irrelevant middle content rather than trusting the model to ignore it. Prefer ranked, deduplicated retrieval over dumping whole files.

## Mechanic three: constraints decay before the window fills

The two complaints heavy users report most often are the same mechanic seen from opposite sides. The first: **somewhere around thirty exchanges, a long session starts breaking rules it was given at the top** — and rules it established for itself. There is no warning and no error; the code simply stops matching the constraints, and you find out at review. The second: after an automatic compaction, **the session behaves like a new agent that has never seen the feature**, usually at the worst possible moment, halfway through one.

Neither is forgetting in any interesting sense. In the first case the constraint is still in the transcript, but it is no longer near either edge of the window, so it competes with two hundred turns of newer material — mechanic two, arriving on a schedule. In the second case the constraint is not in the transcript at all: the summary kept the plot and dropped the rules. Which means both have the same fix, and it is not a bigger window. **Anything that must hold for the whole session belongs in a file that gets re-injected, not in a message that gets buried or summarised.**

> **Warning — The diff that fixes nothing**
>
> The expensive version of this failure is a fabricated file. An agent that is deep in a drifted context can invent a path or a function, and once invented it becomes context for everything after it: later turns reason about the fiction as though it were the codebase, and you get a clean, plausible, well-argued diff against code that does not exist. Nothing errors. The whole refactor makes sense on paper.
>
> The tell is that it never quotes anything. **Require the agent to read before it writes** — “quote the current implementation before you change it”, or an `@` mention that puts the real file in context — and close the loop with a command that fails loudly when the target is missing. A hallucinated file survives review; it does not survive `npm test`.

## The compaction survival map

As context fills, Claude Code clears older tool outputs first, then summarises the conversation. What survives depends entirely on _how each mechanism was loaded_ — and this table is the single most useful thing in the chapter.

| Mechanism | After `/compact` |
| --- | --- |
| System prompt / output style | **Unchanged** — not part of message history |
| Project-root `CLAUDE.md`, auto memory | **Re-injected** from disk |
| Rules with `paths:`, nested `CLAUDE.md` | **Lost** until a matching file is read again |
| Skill descriptions (startup index) | **Not re-injected** |
| Invoked skill bodies | **Re-injected** — 5K tokens per skill, 25K total, oldest dropped first |
| Instructions you typed in chat | **Summarised away.** Put persistent rules in a file, not in a message. |

The practical rule follows directly: **if an instruction must survive compaction, it belongs in project-root `CLAUDE.md`** — not in a nested file, not in a path-scoped rule, and certainly not in something you said forty turns ago.

## Steering what the summary keeps

Two levers. A standing one, in `CLAUDE.md`:

```
## Compact Instructions
When compacting, preserve: current task goal, files being edited,
test commands, and open decisions.
Drop: exploratory reads, superseded plans, verbose tool output.
```

And a per-compaction one, from the prompt:

```
/compact focus on the auth flow and files under src/auth/
```

## Targeted compaction with the rewind menu

`/compact` summarises everything. The rewind menu (`Esc Esc` on an empty input, or `/rewind`) lets you compress one _side_ of a chosen message instead:

| Option | Effect |
| --- | --- |
| **Summarize from here** | Selected message and everything after it become a summary. Earlier context stays in full detail. Use to discard a side quest. |
| **Summarize up to here** | Everything before the selected message becomes a summary. Recent work stays intact. Use to compress setup discussion. |

Both leave the original messages in the session transcript, so Claude can still reach for detail if needed. Highlight a summarize option with the arrow keys and type instructions inline where the row reads **add context (optional)** to steer what it keeps.

## The five moves, ranked

1. **`/clear` between unrelated tasks**

   Resets to zero tokens. The previous conversation stays available in `/resume` — `/clear` resets the window, it does not delete the session. This is the single cheapest, most-skipped move in the book.

2. **Delegate heavy reads to a subagent**

   The subagent runs in its own window and returns a summary. Reading an MCP server’s API reference can cost tens of thousands of tokens; done in a subagent, your window pays for the conclusion only. → Ch. 13

3. **Be specific so Claude reads fewer files**

   “Fix the bug in `auth.ts`“ costs a fraction of “fix the login bug”. Scope investigations narrowly or they become the infinite-exploration failure mode.

4. **`/compact` proactively, with instructions**

   Before the window sprawls, not after. Naming what to preserve is the difference between a useful summary and a lossy one.

5. **Plan outside, paste the result in**

   Do exploratory back-and-forth somewhere cheap, then inject only the final plan — not the dead ends that produced it.

> **Note — The cheap question escape hatch**
>
> `/btw <question>` asks a side question whose answer appears in a dismissible overlay and **never enters conversation history**. Use it for “what does this flag do again?” mid-task instead of spending permanent context on it.

> **Key — The claim this chapter is making**
>
> Context engineering is not housekeeping — it is the discipline that decides output quality, and it holds model-agnostically. JetBrains Research measured the same practice on SWE-bench Verified cutting coding-agent cost by **over 50%**. Treat the window as a budget you spend deliberately, not a bucket you fill.
