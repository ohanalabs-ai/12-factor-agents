# Factor 10 — Small, focused agents (EGM)

Upstream text: [`content/factor-10-small-focused-agents.md`](../content/factor-10-small-focused-agents.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

## Elephant

**Requirement.** Build agents that own one narrow responsibility and take roughly 3-10 steps (up to about 20 at the outside, per the talk), composed by mostly deterministic software, instead of one
monolith that tries everything.

**Why.** Context grows with steps and models lose focus as it does. Narrow scope keeps context short, makes failures local and testable, and gives each
agent a small blast radius. Scope can grow later as reliability allows.

**Observable evidence.** Many small agent definitions each with a one-sentence purpose; narrow tool lists; observed step counts in traces; an orchestrator
delegating to specialists; read-only roles separate from write roles.

**Smallest implementation.** One agent per job, with its own instructions, tool allow-list and an eval; a thin orchestrator.

**Anti-patterns.** One super-agent with every tool; a prompt of thousands of words covering unrelated jobs; hundreds of steps per run.

**What would change this.** If measured reliability stays high at larger scope (evals, step counts, error rates), widen the agent deliberately. Evidence:
eval pass rate at the larger scope.

## Goldfish

Reviewer inputs: agent definitions (instructions, tools), trace step counts.

| ID | Question | Evidence to look for | PASS | PARTIAL | FAIL |
|---|---|---|---|---|---|
| F10.1 | Does each agent have a one-sentence purpose that excludes other jobs? | description field | all | most | a catch-all agent |
| F10.2 | Is each agent's tool list limited to its purpose? | tools/allow-list per agent | narrow | shared broad list | all tools |
| F10.3 | Are median steps per run about 3-10 and p95 about 20 or less? | trace statistics | yes | unknown but plausible | runs of 50+ steps are normal |
| F10.4 | Are read-only investigation and write/act roles separated? | separate agents or identities | separated | same agent, gated | mixed |
| F10.5 | Does an orchestrator delegate rather than do everything inline? | delegation calls, sub-agent config | yes | orchestrator also executes | none |
| F10.6 | Does each agent have its own eval or acceptance test? | per-agent eval | each | some | none |

## From the talk

Source: Dex Horthy's talk, [video](https://www.youtube.com/watch?v=8kMaTybvDUw); timestamps are from its caption track. Paraphrase, not a transcript.

- [00:12:24](https://www.youtube.com/watch?v=8kMaTybvDUw&t=744s) What works in practice is the micro agent: a mostly deterministic DAG with small agent loops of about 3 to 10 steps ([00:12:35](https://www.youtube.com/watch?v=8kMaTybvDUw&t=755s)).
- [00:12:39](https://www.youtube.com/watch?v=8kMaTybvDUw&t=759s) Anecdote: their deploy bot. The pipeline is deterministic CI/CD code until a PR is merged and tests pass on development; then a model is told "get this deployed", proposes front end, a human redirects to back end first, and control returns to deterministic code (end-to-end test against prod, or a rollback agent) ([00:13:13](https://www.youtube.com/watch?v=8kMaTybvDUw&t=793s)).
- [00:13:30](https://www.youtube.com/watch?v=8kMaTybvDUw&t=810s) "100 tools, 20 steps, easy": manageable context and clear responsibilities.
- [00:13:40](https://www.youtube.com/watch?v=8kMaTybvDUw&t=820s) If models improve, expect to start deterministic and sprinkle LLMs in until an agent runs the whole endpoint; still engineer for quality. NotebookLM's advice is quoted as: find something at the edge of what the model does reliably, then engineer reliability around it ([00:14:02](https://www.youtube.com/watch?v=8kMaTybvDUw&t=842s)).
- Correction: our Elephant says "roughly 3-20 steps". The talk's core range is 3 to 10, with 20 as an upper example. The requirement is updated; treat more than about 20 steps per agent as a flag.
