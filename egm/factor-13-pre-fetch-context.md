# Factor 13 — Appendix: pre-fetch all the context you might need (EGM)

Upstream text: [`content/appendix-13-pre-fetch.md`](../content/appendix-13-pre-fetch.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

Note: upstream presents this as an appendix ("Factor 13"), not one of the twelve.

## Elephant

**Requirement.** If the model will very likely need a piece of information, fetch it deterministically before the call and place it in context, instead of
spending a model round trip to ask for it. Let the model do the hard part: using the result.

**Why.** Each tool round trip costs latency, tokens and a chance to go wrong. Predictable lookups (git tags, the open PR, recent deploys, the caller's
profile) are better done by code.

**Observable evidence.** A pre-step that gathers fixed inputs before the first model call; prompts that receive the data, not a "go fetch" instruction;
traces where the first model turn already has the context.

**Smallest implementation.** A `gather()` function called before the loop for the 2-3 inputs used in more than ~80% of runs.

**Anti-patterns.** Always starting with the same three read-only tool calls the model must re-derive; pre-fetching large data "just in case" (violates
factor 3); pre-fetching with elevated credentials.

**What would change this.** If trace data shows the inputs are used in few runs, do not pre-fetch (token cost). Evidence: usage frequency from traces.

## Goldfish

Reviewer inputs: traces of 10 runs, the pre-loop code or harness hooks.

| ID | Question | Evidence to look for | PASS | PARTIAL | FAIL |
|---|---|---|---|---|---|
| F13.1 | Are there inputs fetched deterministically before the first model call? | pre-step code, startup hooks | yes | via a first scripted tool call by the model | none |
| F13.2 | Do traces show the same lookups repeated as the first tool calls of most runs? | tool-call frequency by position | no repeated opening calls | a few | most runs open identically |
| F13.3 | Is pre-fetched data bounded and trimmed (factor 3)? | size limits | yes | partly | dumps |
| F13.4 | Does pre-fetch use least-privilege read-only credentials? | identity used | yes | shared identity | elevated |

## From the talk

Source: Dex Horthy's talk, [video](https://www.youtube.com/watch?v=8kMaTybvDUw); timestamps are from its caption track. Paraphrase, not a transcript.

- Not covered. The talk does not mention pre-fetching context or the appendix. Nothing in this file relies on it.
