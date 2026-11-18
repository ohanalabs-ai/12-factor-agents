# Factor 06 — Launch, pause, resume with simple APIs (EGM)

Upstream text: [`content/factor-06-launch-pause-resume.md`](../content/factor-06-launch-pause-resume.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

## Elephant

**Requirement.** An agent is a program: it can be launched, queried, paused and resumed through simple APIs by users, apps, pipelines and other agents.
Resumption works from an external trigger (a webhook) without deep coupling to the orchestrator, including between tool *selection* and tool *execution*.

**Why.** Long operations and human decisions take minutes to days. If waiting means holding a process open, any restart loses the run and any high-stakes
action cannot be reviewed before it runs.

**Observable evidence.** `launch`, `status`, `pause/cancel`, `resume(event)` endpoints or calls; a run id; a persisted wait state; a resume test that kills
the process and continues.

**Smallest implementation.** `POST /runs`, `GET /runs/{id}`, `POST /runs/{id}/events`, with the state from factor 5 stored durably.

**Anti-patterns.** `while True: sleep()` waiting on a human; resume that restarts from scratch; pause only possible between whole tasks, not before a tool
call executes.

**What would change this.** Short stateless tasks (seconds) do not need pause; the factor then reduces to a launch/cancel API. Evidence needed: maximum
run time and absence of human gates.

## Goldfish

Reviewer inputs: the run API or protocol (A2A, REST, queue), a restart test result, the longest run duration.

| ID | Question | Evidence to look for | PASS | PARTIAL | FAIL |
|---|---|---|---|---|---|
| F06.1 | Is there a documented API to launch a run and get a run id? | endpoint, A2A `message/send`, CLI | yes | launch only through a UI | no |
| F06.2 | Can a run's status be queried by id? | `tasks/get`, `GET /runs/{id}` | yes | via logs only | no |
| F06.3 | Can a run be paused or cancelled by an external caller? | cancel/pause call, input-required state | both | cancel only | neither |
| F06.4 | Can a paused run be resumed by an event (webhook, message) after the process restarted? | resume test, persisted wait | demonstrated | works only while the process lives | restart loses the run |
| F06.5 | Can execution stop between tool selection and tool execution for review? | approval-required flag, pending-call state | yes | per-run only, not per-call | no |
| F06.6 | Is the longest-living wait bounded and surfaced (timeouts, visible state)? | max-run setting, blocked status | bounded and visible | bounded, silent | unbounded |

## From the talk

Source: Dex Horthy's talk, [video](https://www.youtube.com/watch?v=8kMaTybvDUw); timestamps are from its caption track. Paraphrase, not a transcript.

- [00:07:44](https://www.youtube.com/watch?v=8kMaTybvDUw&t=464s) Put the agent behind a REST API or an MCP server; a normal request loads the context window into the LLM.
- [00:07:59](https://www.youtube.com/watch?v=8kMaTybvDUw&t=479s) Let the agent call a long-running tool, interrupt the workflow, store the serialised context; when the work finishes it calls back with the state id and result, the state is loaded, the result appended and sent back to the model. "The agent doesn't even know that things happened in the background" ([00:08:20](https://www.youtube.com/watch?v=8kMaTybvDUw&t=500s)).
- [00:08:23](https://www.youtube.com/watch?v=8kMaTybvDUw&t=503s) "Agents are just software", and this is what unifies factors 5, 6 and 8.
