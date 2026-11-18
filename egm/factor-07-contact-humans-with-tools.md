# Factor 07 — Contact humans with tool calls (EGM)

Upstream text: [`content/factor-07-contact-humans-with-tools.md`](../content/factor-07-contact-humans-with-tools.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

## Elephant

**Requirement.** Asking a human (clarification, approval, "done for now") is a first-class structured intent the model emits, like any tool call, and the
runtime treats it as a reason to leave the loop and wait. The reply returns later as an event.

**Why.** The first token of a response is a high-stakes choice between prose and a structured call. Making "ask a human" an explicit intent gives specific,
typed human contact (who, what, options), enables agent-initiated (outer-loop) workflows, supports several humans and agent-to-agent requests, and makes
the wait durable when combined with factor 6.

**Observable evidence.** Intents such as `request_clarification`, `request_approval`, `done_for_now`; routing to a channel; a human answer recorded as an
event; a decision identity and timestamp; rejection handled.

**Smallest implementation.** Two intents (`ask_human`, `request_approval`) and a handler that posts to a channel, stores the pending item and resumes on
the reply.

**Anti-patterns.** Approvals as chat prose the agent can ignore; a human gate that is only a prompt instruction; approvals that do not record who decided.

**What would change this.** If a platform enforces approval at the tool-call layer (not model-chosen), that satisfies the safety purpose; the
model-chosen clarification intent is then a separate, optional improvement.

## Goldfish

Reviewer inputs: approval/HITL configuration, a trace containing a human decision, the list of high-stakes tools.

| ID | Question | Evidence to look for | PASS | PARTIAL | FAIL |
|---|---|---|---|---|---|
| F07.1 | Is there a structured way for the agent to ask a human (not only free text)? | intent/tool/state such as input-required with typed payload | typed request | free-text only | no way |
| F07.2 | Are high-stakes actions gated by a human decision enforced outside the prompt? | `requireApproval`-style flag, approval workflow, PR review | enforced | instruction only | none |
| F07.3 | Is the human decision recorded (who, when, approve/reject, reason) as an event? | decision record, audit log | all fields | decision without identity or reason | not recorded |
| F07.4 | Does a rejection change the agent's behaviour safely (no leak, no retry loop)? | reject test, bounded retry | tested | handled but untested | not handled |
| F07.5 | Can the agent start a conversation with a human (event/cron-triggered), not only reply? | outer-loop trigger plus contact channel | yes | notifications only | no |
| F07.6 | Can the same mechanism address another agent? | agent-to-agent request through the same intent/protocol | yes | separate mechanism | no |

## From the talk

Source: Dex Horthy's talk, [video](https://www.youtube.com/watch?v=8kMaTybvDUw); timestamps are from its caption track. Paraphrase, not a transcript.

- [00:11:17](https://www.youtube.com/watch?v=8kMaTybvDUw&t=677s) Flagged as subtle: most builders avoid the choice made at the very first output token, tool call versus message to a human. Making "I am done", "I need clarification" or "I need a manager" a natural-language-shaped intent moves that decision onto the first token, in a form the model handles well ([00:11:43](https://www.youtube.com/watch?v=8kMaTybvDUw&t=703s)).
- [00:11:55](https://www.youtube.com/watch?v=8kMaTybvDUw&t=715s) Enables outer-loop agents (agent-initiated work). He defers to a separate post on this.
- [00:12:56](https://www.youtube.com/watch?v=8kMaTybvDUw&t=776s) Anecdote (see factor 10): the deploy bot proposes "deploy the front end", a human replies "back end first", and the natural-language reply becomes the next JSON step; approval then resumes the loop.
- [00:16:25](https://www.youtube.com/watch?v=8kMaTybvDUw&t=985s) Talk-only: he mentions work on a protocol for agents to contact humans (caption reads "A2 protocol", possibly a caption error). Not evidence for a requirement here.
