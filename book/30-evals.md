# Evals

_V · Economics · Chapter 30_

You cannot optimise what you cannot score. Every lever in the last three chapters makes an agent cheaper or faster; none of them tells you whether it still ships correct work. A **40% token cut that drops task completion is not an optimisation win** — it is a regression you shipped with a nice number attached.

- **LLM-as-a-judge** — Using a model with a fixed rubric to score another model’s output or trajectory, instead of exact-match string metrics.
- **G-Eval** — The judge writes evaluation steps, then scores (often 1–5). Highest rank correlation with human ratings among common NLG metric families in Confident AI’s survey.
- **DAG** — A deterministic decision tree: objective criteria with clear success conditions, walked as yes/no branches rather than a free-form score.

## Which scorer for which question

| Family | Use when | Failure mode |
| --- | --- | --- |
| Statistical (BLEU, ROUGE) | A closed reference text exists | Semantically blind |
| Model-based, non-LLM (BERTScore, NLI) | Soft lexical or semantic overlap | Capped by training data |
| LLM-as-a-judge (G-Eval, DAG, QAG) | Subjective, multi-turn, or rubric-heavy work | Probabilistic; needs rubric discipline |

- **CLEAR SUCCESS CONDITION** — DAG — yes/no branches
- **SUBJECTIVE QUALITY** — G-Eval — rubric-driven score
- **FACT-CHECKING AGAINST SOURCES** — QAG — closed yes/no questions
- **HALLUCINATION SNIFF TEST** — SelfCheckGPT — sample and check consistency

## The five-metric rule

Cap the pipeline at roughly five: enough to catch regressions, few enough that humans still read the failures. One or two custom metrics for your use case, plus two or three generic architecture metrics.

| Slot | Agent default | What it catches |
| --- | --- | --- |
| Custom #1 | Task completion — pass/fail against a verifiable condition | “Looks done” that isn’t |
| Custom #2 | Plan adherence or a rubric score | Drift from the agreed steps |
| Generic | Tool correctness | Wrong tool, wrong args, invented APIs |
| Generic | Faithfulness / citation (if RAG) | Answer ignores retrieved context |
| Generic | **Cost proxy** — tokens or $ per _successful_ task | Quality held while spend exploded — or the reverse |

> **Key — You already run one of these**
>
> Claude Code’s `/goal` evaluator _is_ custom metric #1 in production: a separate model reading a completion condition and a transcript, returning a verdict. That is LLM-as-a-judge with a hard gate, not a vibes check — and the same property that makes a good `/goal` condition makes a good eval. → Ch. 22

## A judge prompt you can run today

**Minimal three-score judge · run on a held-out set**

```
You are an eval judge. Score ONLY against the rubric. No coaching.

Rubric:
1. Task completion (0 or 1): did the transcript meet the completion condition?
2. Tool correctness (0 or 1): every tool call used a real tool name and plausible args?
3. Plan adherence (1-5): how closely did the trajectory match the plan steps?

Completion condition:
{{COMPLETION_CONDITION}}

Plan steps:
{{PLAN_STEPS}}

Transcript:
{{TRANSCRIPT}}

Reply as JSON:
{"task_completion":0,"tool_correctness":0,"plan_adherence":1,"notes":"..."}
```

Two rules that decide whether the numbers mean anything. Run the judge on a **held-out set**, not the examples you tuned prompts against — otherwise you are measuring memorisation. And **use a cheaper judge tier for volume**, reserving the frontier tier for contested failures: a judge that costs as much as the work defeats the purpose. → Ch. 27

## Wiring it into the loop

1. **Pick one agent path**

   Not the whole system. One path with a real user behind it.

2. **Write one verifiable completion condition for it**

   Same three parts as a `/goal` condition: measurable end state, stated check, constraints. If you cannot write one, you do not have a task — you have a wish.

3. **Wire the three-score judge, then add the two remaining slots**

   A cost proxy and a faithfulness or citation slot bring you to five.

4. **Fail the build when task completion regresses**

   This is the step that converts an eval from a dashboard into a gate. A metric nobody blocks on is a metric nobody reads.

> **Warning — Log cost next to quality, always on the same row**
>
> **$ per successful task** is the only cost number that cannot lie to you. Tokens per run goes down when the agent gives up early. Cost per run goes down when quality does. Cost per _successful_ task is the metric that catches both — and it is the one that keeps the caching and trimming decisions in Chapter 29 honest.
