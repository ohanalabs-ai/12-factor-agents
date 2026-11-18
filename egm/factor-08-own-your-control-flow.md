# Factor 08 — Own your control flow (EGM)

Upstream text: [`content/factor-08-own-your-control-flow.md`](../content/factor-08-own-your-control-flow.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

## Elephant

**Requirement.** The loop that decides what happens after each model output is code the team owns. It decides per intent whether to continue immediately,
break out and wait for a human or long task, summarise, cache, rate-limit, judge or compact.

**Why.** Different intents have different risk and latency. Without owned control flow the only options are blocking in memory, limiting the agent to
low-stakes work, or hoping. Interrupting between selection and invocation is the key capability.

**Observable evidence.** An explicit loop with per-intent branches (continue, wait, approve, stop); counters and limits (steps, errors, spend); durable wait;
logging, tracing and metrics in the loop.

**Smallest implementation.** A `while` loop with a `match` on intent, a step limit, and `return` paths for wait/approve states that persist state.

**Anti-patterns.** "Loop until done" with no step cap; control flow described only in prompt prose ("first do X, then Y"); no way to stop an agent mid-run.

**What would change this.** For well-understood multi-step jobs, a deterministic pipeline with LLM steps beats an agent loop. Evidence: a fixed step order
that never varies means the control flow should be code, not model choice.

## Goldfish

Reviewer inputs: the orchestration code, harness settings, a runaway-run test or its absence.

| ID | Question | Evidence to look for | PASS | PARTIAL | FAIL |
|---|---|---|---|---|---|
| F08.1 | Is there a hard cap on steps or tool calls per run, enforced in code or runtime? | max turns, max-run-seconds, budget limit | enforced | documented, not enforced | none |
| F08.2 | Is the step order that never varies implemented as code rather than prompt prose? | pipeline code or workflow definition for fixed sequences | fixed parts are code | mixed | all in prompt |
| F08.3 | Does at least one intent exit the loop to wait (human, event, long job)? | wait states in code | yes | only by ending the run | no |
| F08.4 | Are cost or token budgets enforced per run or per day? | budget gate, spend events | enforced | monitored only | none |
| F08.5 | Is each iteration observable (logs, traces, metrics with run id)? | trace spans per step | yes | coarse (per run) | no |
| F08.6 | Is there a deterministic kill switch an operator can use mid-run? | cancel API, pause agent | yes | stop the process only | no |

## From the talk

Source: Dex Horthy's talk, [video](https://www.youtube.com/watch?v=8kMaTybvDUw); timestamps are from its caption track. Paraphrase, not a transcript.

- [00:05:06](https://www.youtube.com/watch?v=8kMaTybvDUw&t=306s) Code is already a graph; DAG orchestrators (Airflow, Prefect) give reliability by breaking work into nodes. Agents were supposed to remove the need to draw the DAG: give a goal, the model finds the way, and the loop appends to the context until the model says done ([00:05:53](https://www.youtube.com/watch?v=8kMaTybvDUw&t=353s)).
- [00:06:25](https://www.youtube.com/watch?v=8kMaTybvDUw&t=385s) Sharpening: he says this naive loop "doesn't really work", mostly because long context windows hurt reliability on longer workflows. His own anatomy is prompt, switch statement over model JSON, context builder, and a loop that decides when to exit ([00:06:56](https://www.youtube.com/watch?v=8kMaTybvDUw&t=416s)).
- [00:07:14](https://www.youtube.com/watch?v=8kMaTybvDUw&t=434s) What owning it buys: break, switch, summarise and LLM-as-judge in the loop.
- [00:01:06](https://www.youtube.com/watch?v=8kMaTybvDUw&t=66s) Anecdote supporting our "what would change this": a DevOps build agent took hours of prompt tuning to fix an exact step order, when a bash script would have taken about 90 seconds. Not every problem needs an agent.
- Correction to our wording: the factor is not only about pausing. The talk justifies it first by context length and reliability, then by pause/resume.
