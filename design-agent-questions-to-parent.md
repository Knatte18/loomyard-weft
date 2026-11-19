# Every run agent question for the operator goes to the run parent

`agent-questions-to-parent` · Note · feature, loom, shuttle

## What was seen

From a comment on issue #153 (2026-11-19): a run's agents (drift judge, Drift-Expect, perch fixer) put their questions to the operator in their own pane.
The parent sees such a question only when the operator relays it or tells the agent to ask the parent.

## Operator decision (2026-11-19)

Only an orch contacts the operator; an orch is the only session with a Slack relay.
By default, every question a run agent would put to the operator goes to the run's parent orch by creel, with the parent's answer delivered back.
It is the default for every run agent, not an opt-in setting. The parent answers itself or asks the operator.

## To find out

Which agents and stencils still address the operator directly (drift judge, Drift-Expect, perch fixers and any other),
and how the shuttle's held-turn notice already routes a question to the parent; the change closes each remaining path.

Related: `parent-review-code-and-drift` routes review rulings to the parent; this routes the agents' open questions.
