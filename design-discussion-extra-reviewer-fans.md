# Extra reviewer fans at the Discussion perch: parent orch or a named agent

`discussion-extra-reviewer-fans` · Note · feature, loom, burler, config, orch

Imported from [issue #113](https://github.com/Knatte18/loomyard/issues/113).

## Seen

A parent orch that wants an independent review of a Discussion record before the perch has no supported path.
The workaround today:

1. `lym loom pause --after Discussion-Write` in the task worktree.
2. The orch spawns its own read-only agent (here a Fable agent) to review the record.
3. The accepted findings are recorded with `lym loom decision add --by parent`, then `lym loom resume`.

This takes the review outside the perch.
Its findings arrive as parent decisions rather than as review findings, the perch runs after it rather than beside it, and the orch has to settle every finding before the perch starts.

## Wanted

An option, in `loom.yaml` and in a task's `run_settings`, that adds extra reviewer fans to the Discussion perch's fan-out:

- a `parent` fan: the run's parent orch reviews the record;
- an agent fan with a named model and effort, such as a Fable agent at high effort;
- either or both, each its own fan.

The extra fans review at the same time as the Burler-Review lenses.
Their findings join the round's findings like any other lens, and Burler-Fix fixes them in the same round.
No pause, no extra round and no parent decision is needed.

The same option could later apply to the Plan and Code perches.

## Belongs with `parent-reviews-decision-record` (operator, 2026-11-11)

This note and `parent-reviews-decision-record` ask for one thing from two sides: an extra review of the Discussion record by the parent orch or a named agent.
They are designed together, in one Motif.
