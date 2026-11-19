# Send a run agent questions for the operator to the run parent instead

`agent-questions-to-parent` · Note · feature, loom, shuttle

## What was seen

From a comment on issue #153 (2026-11-19): a run's agents (drift judge, Drift-Expect, perch fixer) put their questions to the operator in their own pane.
The parent sees such a question only when the operator relays it or tells the agent to ask the parent.

## Idea

A setting that sends every question an agent would put to the operator to the run's parent by creel instead, with the parent's answer delivered back,
so the parent settles them without the operator relaying.

Related: `parent-review-code-and-drift` routes review rulings to the parent; this routes the agents' open questions.
