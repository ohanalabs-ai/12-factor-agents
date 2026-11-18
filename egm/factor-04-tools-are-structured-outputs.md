# Factor 04 — Tools are structured outputs (EGM)

Upstream text: [`content/factor-04-tools-are-structured-outputs.md`](../content/factor-04-tools-are-structured-outputs.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

## Elephant

**Requirement.** Treat a "tool call" as nothing more than model output that describes what deterministic code should do. The model decides *what*; code
decides *how*, and may do it differently each time (queue it, ask a human, batch it, refuse it).

**Why.** If a tool call is an inevitable immediate function call, there is no seam to insert review, rate limits, caching or approval. The seam between
"the model asked" and "the code did" is where safety and reliability are enforced.

**Observable evidence.** A dispatcher (switch/router) between parsed output and handlers; handlers that can return "deferred", "needs approval" or
"rejected"; policy applied there (allow-lists, scopes, approval).

**Smallest implementation.** A `match intent:` dispatcher where each case may execute, defer, or escalate, plus a log of every intent and its outcome.

**Anti-patterns.** Tool names wired 1:1 to side-effecting functions with no policy layer; an allow-list enforced only in the prompt; the model able to call
anything the server exposes.

**What would change this.** If a gateway enforces scoping and approval for every tool call outside the agent, the seam exists even if the agent code is a
thin loop. Evidence: the gateway policy and a denied-call test.

## Goldfish

Reviewer inputs: dispatcher code or gateway policy, tool allow-lists, denied-call evidence.

| ID | Question | Evidence to look for | PASS | PARTIAL | FAIL |
|---|---|---|---|---|---|
| F04.1 | Is there a layer between model output and side effects that can say no? | dispatcher, gateway policy, MCP authorisation | yes, exercised | exists, never shown denying | none |
| F04.2 | Is the set of callable tools restricted per agent by configuration enforced outside the prompt? | allow-list in agent config / gateway; test that an out-of-list call fails | enforced and tested | configured but the runtime exposes more than listed | prompt-only |
| F04.3 | Can an intent be handled differently from "execute now" (defer, approve, batch)? | at least one tool routed to approval or queue | yes | possible but unused | no |
| F04.4 | Are read and write capabilities separated (different tools, scopes or identities)? | read-only vs write tool sets; separate service accounts | separated | partly (naming only) | mixed |
| F04.5 | Is every intent and its outcome logged with arguments and a correlation id? | structured log or trace per call | yes | outcome only | no |

## From the talk

Source: Dex Horthy's talk, [video](https://www.youtube.com/watch?v=8kMaTybvDUw); timestamps are from its caption track. Paraphrase, not a transcript.

- [00:04:04](https://www.youtube.com/watch?v=8kMaTybvDUw&t=244s) Talk-only analogy: "tool use is harmful" in the way goto was (quoted as a provocation). He is explicit it is not about denying agents access to the world; the harm is the mental image of an alien entity acting on its environment.
- [00:04:42](https://www.youtube.com/watch?v=8kMaTybvDUw&t=282s) What actually happens: the model emits JSON, deterministic code does something, maybe the result is fed back. Parsed JSON goes into a loop or a switch statement; "nothing special about tools, it's just JSON and code" ([00:05:00](https://www.youtube.com/watch?v=8kMaTybvDUw&t=300s)).
- Review heuristic confirmed: the dispatcher is the seam (F04). The talk does not add the gateway-as-seam variation; that remains our interpretation.
