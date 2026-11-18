# 12-factor agents x EGM — an Elephant and a Goldfish per factor

This directory is an OhanaLabs addition to the fork. It does **not** change the upstream content (`../content/`). It turns the factors into an
**evaluator**: for each factor an *Elephant* (first-principles statement) and a *Goldfish* (a fresh-context checklist) in the sense of our
[EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md).

Source status: the factor text is the repository's `content/factor-*.md` and `appendix-13-pre-fetch.md`. The talk
(https://www.youtube.com/watch?v=8kMaTybvDUw) has no transcript in the repository and was not watched; nothing here relies on it.
Factor 12's upstream text is almost empty (two images), so its reading is flagged as interpretation in its file.

| # | Factor | File |
|---|---|---|
| 1 | Natural language to tool calls | [factor-01](factor-01-natural-language-to-tool-calls.md) |
| 2 | Own your prompts | [factor-02](factor-02-own-your-prompts.md) |
| 3 | Own your context window | [factor-03](factor-03-own-your-context-window.md) |
| 4 | Tools are structured outputs | [factor-04](factor-04-tools-are-structured-outputs.md) |
| 5 | Unify execution state and business state | [factor-05](factor-05-unify-execution-state.md) |
| 6 | Launch, pause, resume with simple APIs | [factor-06](factor-06-launch-pause-resume.md) |
| 7 | Contact humans with tool calls | [factor-07](factor-07-contact-humans-with-tools.md) |
| 8 | Own your control flow | [factor-08](factor-08-own-your-control-flow.md) |
| 9 | Compact errors into the context window | [factor-09](factor-09-compact-errors.md) |
| 10 | Small, focused agents | [factor-10](factor-10-small-focused-agents.md) |
| 11 | Trigger from anywhere | [factor-11](factor-11-trigger-from-anywhere.md) |
| 12 | Stateless reducer | [factor-12](factor-12-stateless-reducer.md) |
| 13 | Appendix: pre-fetch context | [factor-13](factor-13-pre-fetch-context.md) |

## How a review runs (Goldfish)

1. The reviewer is given only the agent's code, config, prompts and 5-10 traces (never the author's explanation).
2. For each factor it answers the question IDs (`F01.1` ...) with `pass`, `partial` or `fail` and **one evidence pointer** (file:line, trace id, config
   key). No pointer means the answer is `fail`, or `unknown` when the material was not provided.
3. Scoring: pass = 2, partial = 1, fail = 0, `unknown` excluded and reported. Factor status: **implemented** when at least 80% of answered points are scored
   and no question is `fail`; **not yet** when under 35%; otherwise **partial**. Factor 12 and 13 are reported but excluded from the headline score.
4. The output is a scorecard (`schema` below). The Elephant section of each factor tells the reviewer which variations are legitimate (the "what would
   change this" paragraph) so that an equivalent mechanism is not marked down.

```yaml
agent: <name>
reviewed_at: <iso8601>
material: [<paths or trace ids>]
factors:
  - factor: 7
    status: implemented | partial | not-yet
    score: 9/12
    answers:
      - { id: F07.2, result: pass | partial | fail | unknown, evidence: "<path:line | trace id>" }
headline: <sum of factors 1-11 / max>
```

These checklists are the seed of a customer-facing analyzer and of a certifier/auditor; the question wording is written to be answerable from artifacts.
