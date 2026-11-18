# Factor 12 — Make your agent a stateless reducer (EGM)

Upstream text: [`content/factor-12-stateless-reducer.md`](../content/factor-12-stateless-reducer.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

> Note: the upstream file for this factor is a heading and two images (`1c0-stateless-reducer.png`, `1c5-agent-foldl.png`); the prose says only that it is
> "mostly just for fun". The reading below is derived from the image names (an agent as a fold over events) and from factors 3, 5 and 6. Treat it as
> interpretation, and weigh it less than factors 1-11 when scoring.

## Elephant

**Requirement.** The agent step is a pure function `(state, event) -> state'` (or `-> next intent`): given the same thread it produces the next step, holds
nothing in process memory between steps, and can run on any worker.

**Why.** If all state is in the event thread (factor 5), any process can handle the next event; restarts, scale-out, replay and testing become trivial,
and the agent composes like any function.

**Observable evidence.** A handler that loads state by run id, appends one event, writes back and exits; replay of a stored thread yielding the same next
intent (modulo model nondeterminism); no sticky sessions.

**Smallest implementation.** `handle(run_id, event)`: load thread, `next = decide(thread + event)`, save, return.

**Anti-patterns.** Per-run objects living in a long-lived process; sticky routing; state hidden in files on the worker.

**What would change this.** Where the runtime snapshots the whole sandbox/actor (filesystem and memory) to external storage, durability is achieved by a
different route. Evidence: restore from snapshot on a different node continues correctly. It then satisfies the purpose (restart and relocation
safety) while not being a reducer in the strict sense. Score as PARTIAL.

## Goldfish

Reviewer inputs: the run handler or runtime architecture, a restart/relocation test.

| ID | Question | Evidence to look for | PASS | PARTIAL | FAIL |
|---|---|---|---|---|---|
| F12.1 | Is all information needed for the next step loaded from the stored thread by run id? | handler loads by id | yes | loads thread plus local files | uses in-process objects |
| F12.2 | Can the next step run on a different process or node than the previous one? | stateless workers, relocation test | demonstrated | works via whole-environment snapshot restore | no |
| F12.3 | Can a stored thread be replayed into the decision function in a test? | replay test or fixture | yes | manual only | no |
| F12.4 | Is there no sticky routing or in-memory per-run state between events? | deployment config | none | some caching, safe to lose | required |
