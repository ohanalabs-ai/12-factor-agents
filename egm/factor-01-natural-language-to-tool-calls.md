# Factor 01 — Natural language to tool calls (EGM)

Upstream text: [`content/factor-01-natural-language-to-tool-calls.md`](../content/factor-01-natural-language-to-tool-calls.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

## Elephant

**Requirement.** The model's job at the boundary is to turn a request into a structured, schema-valid call description (a tool name plus typed
arguments). Deterministic code, not the model, performs the call.

**Why (first principles).** A model emits tokens; only code can touch an API. A typed payload is the narrowest interface between them: it can be
validated, logged, diffed, replayed and refused. Free text that code must "interpret" re-introduces a second, untested parser.

**Observable evidence.** A declared tool/function schema per capability; the runtime validates model output against it before acting; traces show
`tool_name + arguments` records, not prose; invalid output is rejected, not executed.

**Smallest implementation.** One typed schema per tool (JSON Schema, Pydantic, BAML, MCP tool definition) and one dispatcher that parses, validates and
calls. No framework required.

**Anti-patterns.** Parsing the model's prose with regexes; letting the model build shell strings or SQL directly; tool arguments typed as one free-text field.

**What would change this.** If a provider's native tool-calling proves less reliable than constrained decoding for a task, the schema stays but the
transport changes. The factor is about the typed boundary, not about any one API.

## Goldfish

Reviewer inputs: the agent's tool definitions, the dispatcher code or MCP server config, and one trace of a tool call.

| ID | Question | Evidence to look for | PASS | PARTIAL | FAIL |
|---|---|---|---|---|---|
| F01.1 | Does every action the agent can take have a declared, typed schema? | tool/MCP definitions with `inputSchema`/parameters; no "run any command" catch-all | all actions typed | typed, but one broad escape hatch (shell, generic HTTP) | actions are free text |
| F01.2 | Is model output validated against that schema before anything executes? | validation call in the dispatcher or the runtime/MCP layer; a rejected-call log line or test | validated, invalid is rejected | validated by the vendor runtime only, not testable by the team | no validation |
| F01.3 | Does a trace of one run show structured `name + args` records for each call? | trace/log sample with tool name and JSON args | present for every call | present for some, or args truncated | prose only |
| F01.4 | Is execution done by code the team owns (or reviews), separate from the model call? | dispatcher/handler code, not the prompt, performs the side effect | clear separation | separation exists but handlers embedded in prompts or skills | model output is executed as-is |
| F01.5 | Are arguments narrow (enums, ids, bounded strings) where the domain allows? | enums or ids in schemas instead of free strings for ids, environments, namespaces | narrow where possible | some narrowing | free strings everywhere |

## From the talk

Source: Dex Horthy's talk, [video](https://www.youtube.com/watch?v=8kMaTybvDUw); timestamps are from its caption track. Paraphrase, not a transcript.

- [00:03:46](https://www.youtube.com/watch?v=8kMaTybvDUw&t=226s) Framed as the most "magical" capability: turning a sentence into JSON. It matters before any loop, switch or tool exists, and it is the one piece you can adopt in an app today without adopting anything else.
- [00:02:21](https://www.youtube.com/watch?v=8kMaTybvDUw&t=141s) Talk-only framing: the factors are software-engineering practice (compared to what Heroku's 12 factors did for cloud apps), not an AI specialty. Review heuristic: do not require agent-framework vocabulary to credit this factor.
