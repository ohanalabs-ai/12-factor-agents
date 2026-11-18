# Factor 05 — Unify execution state and business state (EGM)

Upstream text: [`content/factor-05-unify-execution-state.md`](../content/factor-05-unify-execution-state.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

## Elephant

**Requirement.** Keep the agent's execution state (current step, waiting status, retries) and its business state (what happened: messages, tool calls,
results) in one serialisable record where possible; infer execution state from the event history instead of a parallel state machine.

**Why.** Two sources of truth drift. One append-only thread gives free serialisation, debugging, resume from any point, forking, and a human-readable
view. The factor allows exceptions (session ids, credentials) but says to minimise them.

**Observable evidence.** A persisted thread/event log per run; "where is it now" derivable from that log; resume loads the thread and continues; the log
can be rendered for a human.

**Smallest implementation.** An append-only list of typed events stored per run id, plus `next_step(events)`.

**Anti-patterns.** Step counters and flags in process memory; a status column that disagrees with the transcript; state that exists only in a vendor
session that cannot be exported.

**What would change this.** For long, branching workflows an explicit workflow engine may justify separate execution state. Evidence: a measured
failure of event-derived state (replay cost, ambiguity). Absent that, unify.

## Goldfish

Reviewer inputs: the persistence layer, the run record, a restart test.

| ID | Question | Evidence to look for | PASS | PARTIAL | FAIL |
|---|---|---|---|---|---|
| F05.1 | Is each run's history persisted as a record the team can read outside the running process? | DB rows, event log, snapshot store, issue/comment thread | yes | persisted only inside a vendor session | memory only |
| F05.2 | Can the current step or wait reason be derived from that history without a second store? | derivation code or a statement of what extra fields exist | derived | small extra flags, documented | a separate state machine that can disagree |
| F05.3 | Can a run be forked or replayed from an earlier point? | fork/replay feature, snapshot restore | yes | restore only to the latest point | no |
| F05.4 | Can a human read the whole run without special tools? | markdown/UI rendering of the thread | yes | needs developer tooling | no |
| F05.5 | Is non-context state (credentials, session ids) minimised and kept out of the model's view? | list of side-channel state | minimal and documented | some, undocumented | large hidden state |
