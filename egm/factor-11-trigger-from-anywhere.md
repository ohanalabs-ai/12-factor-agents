# Factor 11 — Trigger from anywhere, meet users where they are (EGM)

Upstream text: [`content/factor-11-trigger-from-anywhere.md`](../content/factor-11-trigger-from-anywhere.md) (unchanged). Everything below is OhanaLabs interpretation (Elephant and Goldfish
per the [EGM process](https://github.com/ohanalabs-ai/agentic-process/blob/main/docs/02-egm-first-principles-process.md)); the upstream text is the only source of what the factor says.

## Elephant

**Requirement.** Agents can be started by humans from the channels they already use (chat, email, SMS, issue trackers) and by non-humans (events, crons,
outages), and answer in the same channel, looping in people at critical points.

**Why.** Real work starts in many places; outer-loop agents that run for minutes then ask a human unlock higher-stakes work because approval is one message
away. This depends on factors 6 and 7.

**Observable evidence.** More than one trigger type wired to the same launch API; a channel adapter that maps messages to run events and back; the
originating channel stored on the run; event/cron triggers.

**Smallest implementation.** The launch API from factor 6 plus one webhook adapter per channel and one scheduler.

**Anti-patterns.** An agent reachable only from its own bespoke UI; channel logic inside the agent; triggers that bypass authentication and audit.

**What would change this.** For an internal back-office agent one channel may be right. Evidence: users and trigger sources are all in that channel.

## Goldfish

Reviewer inputs: trigger configuration, channel adapters, auth on each trigger.

| ID | Question | Evidence to look for | PASS | PARTIAL | FAIL |
|---|---|---|---|---|---|
| F11.1 | Is there more than one way to start a run, all converging on the same launch API? | list of triggers | 2 or more | 1 plus a documented plan | 1 bespoke path |
| F11.2 | Is at least one trigger non-human (event, schedule, alert, merge)? | cron/event config | yes | manual replays only | no |
| F11.3 | Does the reply return through the originating channel? | channel stored on run, outbound adapter | yes | different channel | no reply path |
| F11.4 | Is every trigger authenticated and attributed to an identity in the run record? | auth config, run record has actor | yes | authenticated, not attributed | open trigger |
| F11.5 | Is channel logic outside the agent's core instructions? | adapter separate from prompts | yes | mixed | embedded |
