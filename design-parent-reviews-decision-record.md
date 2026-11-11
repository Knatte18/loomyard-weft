# Optional extra review of the Discussion record by the parent orch or a named agent

`parent-reviews-decision-record` · Task · feature, loom, orch, config, burler · running · Open-Window

Imported from [issue #87](https://github.com/Knatte18/loomyard/issues/87) and [issue #113](https://github.com/Knatte18/loomyard/issues/113).

## Idea

An extra review of the Discussion decision record by the parent orch or by a named agent, before Plan starts.
Two notes asked for it from two sides and are designed together, in one Motif (operator, 2026-11-11).

## Optional, off per hub and per task (operator, 2026-11-11)

Every such extra review is optional.
Each can be switched off for a hub, in `loom.yaml`, and for a task, in its `run_settings`, and a hub or task that does not switch one on runs without it.
The Motif decides the default and how a task overrides the hub's setting.

## Seen: the parent reviews the record (issue #87)

In hub `dop`, the operator has the run's PR-Review reviewer ask the parent orch for a review, and finds it very useful: the orch knows what the run needs and what the operator wants.
But at PR time much is too late to change cheaply.
The plan is too large to be a good review point for the parent, and the plan is only meant to realize the Discussion decision record.
An older LoomYard version, and millhouse today, let the parent review the Discussion record.

Example: tests-tier1 met a scope question (whether to fix EFCoreStore's loss of SimulationResults) in Plan-Burler-Fix, and the orch had to record a parent decision mid-run; reviewing the record would have settled it before the plan was written.

### Expected

A setting, per hub and per task, that makes the run hand its Discussion decision record to the parent orch for review before Plan starts.
The orch answers approved or sends findings; findings go back to Discussion as a review round does, and Plan starts only on approval.
It complements the PR-time review that has landed (`639e4fab`): the `loom.yaml` key `pr_review_ask_parent`, default `false`, open to a task's `run_settings`, makes the PR-Review row mail the parent orch a `[pr-review request]`.
This setting could replace it for most runs.

### Constraint (operator, 2026-11-08)

A parent orch is often very busy with coordination (the LoomYard hub's orch especially), which is why the parent review was dropped before.
The design should keep the parent's load small: the review is opt-in per task, so a busy orch's hub leaves it off;
the orch can hand the review to a fork of itself, which keeps its knowledge of the run but keeps the review out of the orch's main thread;
and a deadline lets the run go on to Plan without the parent's review, recording that it did.

## Seen: no supported path for an independent review (issue #113)

A parent orch that wants an independent review of a Discussion record before the perch has no supported path.
The workaround today:

1. `lym loom pause --after Discussion-Write` in the task worktree.
2. The orch spawns its own read-only agent (here a Fable agent) to review the record.
3. The accepted findings are recorded with `lym loom decision add --by parent`, then `lym loom resume`.

This takes the review outside the perch.
Its findings arrive as parent decisions rather than as review findings, the perch runs after it rather than beside it, and the orch has to settle every finding before the perch starts.

### Wanted

An option, in `loom.yaml` and in a task's `run_settings`, that adds extra reviewer fans to the Discussion perch's fan-out:

- a `parent` fan: the run's parent orch reviews the record;
- an agent fan with a named model and effort, such as a Fable agent at high effort;
- either or both, each its own fan.

The extra fans review at the same time as the Burler-Review lenses.
Their findings join the round's findings like any other lens, and Burler-Fix fixes them in the same round.
No pause, no extra round and no parent decision is needed.

The same option could later apply to the Plan and Code perches.

## Open for the Discussion

Whether the parent's review is a fan beside the lenses (the second note) or a step before Plan with its own round (the first note), or both, and how a deadline and a fork of the orch fit either.
