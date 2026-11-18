# Factor 03 — Own your context window (EGM)

Upstream text: [`content/factor-03-own-your-context-window.md`](../content/factor-03-own-your-context-window.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

## Elephant

**Requirement.** The team decides what goes into the model's context on every turn and in what format: instructions, retrieved documents, history, tool
results, errors, memory. Everything is context engineering; the model is a stateless function of its input.

**Why.** Quality, cost and safety are all functions of the input. Attention is finite and context grows with every step; what is not selected, trimmed
or filtered by the team is selected by defaults. Safety also lives here: sensitive data must be filtered before the model sees it.

**Observable evidence.** A single place where context is assembled; history entries typed as events; trimming/summarisation/compaction policy; a filter
for sensitive fields; the ability to view the full context of any past turn.

**Smallest implementation.** A function `build_context(events) -> text` the team owns, a token budget with a stated drop policy, and a redaction step.

**Anti-patterns.** Appending the raw transcript forever; stuffing whole tool outputs (logs, JSON dumps) into history; letting a framework decide what
history survives; resolved errors left in context.

**What would change this.** If the platform exposes a managed, inspectable context policy with budgets and redaction, using it can satisfy the factor.
Evidence needed: the policy is configurable and the effective context is auditable.

## Goldfish

Reviewer inputs: the context-assembly code or the harness configuration, and one long trace.

| ID | Question | Evidence to look for | PASS | PARTIAL | FAIL |
|---|---|---|---|---|---|
| F03.1 | Is there one identifiable place where the model's input is assembled, owned by the team? | a function/module or a harness config section (system prompt, skills, memory, tool list) | yes | assembled across several vendor defaults, but each is configurable | not locatable |
| F03.2 | Is there a stated policy for what happens when context grows (budget, truncation, summary, compaction)? | token budget constants, summariser, doc stating the policy | policy and implementation | policy only, or implementation relies on vendor auto-compaction | none |
| F03.3 | Are large tool results shaped before entering context (fields selected, truncated, summarised)? | wrapper that trims MCP/tool output; limits on logs or lists | shaped | shaped for some tools | raw dumps |
| F03.4 | Is sensitive data filtered before it reaches the model? | redaction step, deny-list, secret-name-only reads | enforced in code or gateway | by instruction only ("do not print secrets") | none |
| F03.5 | Can a reviewer reconstruct the exact context of a given past turn? | stored events plus a render function, or full request logging | yes | partially (inputs but not final render) | no |
| F03.6 | Is the format chosen for density/clarity (typed events, delimiters), not just inherited message lists? | custom event format or documented reason for the default | custom or justified default | default, undocumented | default, and traces show duplicated or bloated context |

## From the talk

Source: Dex Horthy's talk, [video](https://www.youtube.com/watch?v=8kMaTybvDUw); timestamps are from its caption track. Paraphrase, not a transcript.

- [00:09:28](https://www.youtube.com/watch?v=8kMaTybvDUw&t=568s) Own how the context window is built: the standard messages format is one option, but you can fold the whole event history into a single user or system message and ask "what happens next"; the thread model and its serialisation are yours.
- [00:10:12](https://www.youtube.com/watch?v=8kMaTybvDUw&t=612s) "Everything in making agents good is context engineering": prompt, memory, RAG and history are all just ways to get the right tokens in. Look at every token and optimise density and clarity.
- [00:06:49](https://www.youtube.com/watch?v=8kMaTybvDUw&t=409s) Talk-only (evidence for F03 limits): long context can be accepted by the API, yet nobody disputes that limiting the tokens gives tighter, more reliable results.
