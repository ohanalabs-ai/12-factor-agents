# Factor 10 — Small, focused agents (EGM)

Upstream text: [`content/factor-10-small-focused-agents.md`](../content/factor-10-small-focused-agents.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

## Elephant

**Requirement.** Build agents that own one narrow responsibility and take roughly 3-20 steps, composed by mostly deterministic software, instead of one
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
| F10.3 | Are median and p95 steps per run in the low tens or less? | trace statistics | yes | unknown but plausible | runs of 50+ steps are normal |
| F10.4 | Are read-only investigation and write/act roles separated? | separate agents or identities | separated | same agent, gated | mixed |
| F10.5 | Does an orchestrator delegate rather than do everything inline? | delegation calls, sub-agent config | yes | orchestrator also executes | none |
| F10.6 | Does each agent have its own eval or acceptance test? | per-agent eval | each | some | none |
