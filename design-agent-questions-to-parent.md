# Run agents ask their parent orch, and the parent can subscribe to a run reaching a named row

`agent-questions-to-parent` · Task · feature, loom, shuttle, shed · running · Drift-Baseline

## Part 1: agent questions go to the parent

### What was seen

From a comment on issue #153 (2026-11-19): a run's agents (drift judge, Drift-Expect, perch fixer) put their questions to the operator in their own pane.
The parent sees such a question only when the operator relays it or tells the agent to ask the parent.

### Operator decision (2026-11-19)

Only an orch contacts the operator; an orch is the only session with a Slack relay.
By default, every question a run agent would put to the operator goes to the run's parent orch by creel, with the parent's answer delivered back.
It is the default for every run agent, not an opt-in setting. The parent answers itself or asks the operator.

### To find out

Which agents and stencils still address the operator directly (drift judge, Drift-Expect, perch fixers and any other),
and how the shuttle's held-turn notice already routes a question to the parent; the change closes each remaining path.

The landed `parent-review-code-and-drift` routes review rulings to the parent; this routes the agents' open questions.

## Part 2: the parent subscribes to a named row

### What was seen

From a comment on issue #153 (2026-11-19): a parent that wants to brief the agent of a given row as it starts has to poll `lym loom status --watch`.

### Idea

Let the parent subscribe to a run's state, so lym mails it when the run reaches a named producer or state:
for example `lym loom notify <slug> --on Drift-Verify,Drift-Expect,Code-Burler-Fix`, or a `run_settings` key such as `notify_parent_on: [Drift-Verify]`.
The parent can then brief each agent of those rows as it starts.
Discussion chooses between the verb and the key.
