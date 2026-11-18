# Factor 09 — Compact errors into the context window (EGM)

Upstream text: [`content/factor-09-compact-errors.md`](../content/factor-09-compact-errors.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

## Elephant

**Requirement.** When a tool fails, put a compact, useful form of the error back into context so the model can self-correct, with a counter that stops
repeated failure and escalates (to control flow or a human).

**Why.** Models read error messages well, so retry-with-error is cheap self-healing. Unbounded, it spins on the same mistake and fills the context with
noise. The fix is structure (factors 3 and 8) and small agents (factor 10).

**Observable evidence.** Tool errors normalised to a short structure (code, message, hint); a per-tool consecutive-error counter; a threshold that
escalates; resolved errors dropped from later context.

**Smallest implementation.** Catch exceptions in the dispatcher, append `{error, tool, args}` truncated to N characters, increment a counter, escalate at 3.

**Anti-patterns.** Full stack traces in context; silent retry hiding the failure; infinite retry; swallowing errors so the agent believes it succeeded.

**What would change this.** For non-idempotent side effects, retries can be harmful; then the rule becomes "fail visibly and stop" (a legitimate
variation, with the stop recorded). Evidence: the tool is not idempotent.

## Goldfish

Reviewer inputs: error-handling code, a trace containing a failed tool call.

| ID | Question | Evidence to look for | PASS | PARTIAL | FAIL |
|---|---|---|---|---|---|
| F09.1 | Does a failed tool call return a structured error to the model rather than crashing or hiding it? | error event in trace with tool, message | yes | raw error text | crash or silent |
| F09.2 | Is the error text bounded (truncated, normalised)? | length limit, formatter | bounded | partially | raw traces |
| F09.3 | Is there a consecutive-error limit per tool or per run? | counter and threshold | enforced | informal ("try again a few times") | none |
| F09.4 | Does hitting the limit escalate (human, different path, stop) visibly? | blocked status, notification | yes | stops silently | loops |
| F09.5 | For non-idempotent actions, is retry explicitly disabled or guarded? | "no retries" statement, idempotency key | yes | undocumented | blind retry |
| F09.6 | Are resolved errors removed or summarised from later context? | context builder logic | yes | no, but runs are short | accumulates |
