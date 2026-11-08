# Let the parent orch review the Discussion decision record before Plan

`parent-reviews-decision-record` · Note · feature, loom, orch, config

Imported from [issue #87](https://github.com/Knatte18/loomyard/issues/87).

## Seen

In hub `dop`, the operator has the run's PR-Review reviewer ask the parent orch for a review, and finds it very useful: the orch knows what the run needs and what the operator wants.
But at PR time much is too late to change cheaply.
The plan is too large to be a good review point for the parent, and the plan is only meant to realize the Discussion decision record.
An older LoomYard version, and millhouse today, let the parent review the Discussion record.

Example: tests-tier1 met a scope question (whether to fix EFCoreStore's loss of SimulationResults) in Plan-Burler-Fix, and the orch had to record a parent decision mid-run; reviewing the record would have settled it before the plan was written.

## Expected

A setting, per hub and per task, that makes the run hand its Discussion decision record to the parent orch for review before Plan starts.
The orch answers approved or sends findings; findings go back to Discussion as a review round does, and Plan starts only on approval.
It complements #86 (the parent asked for review at PR time), which has landed (`639e4fab`): the `loom.yaml` key `pr_review_ask_parent`, default `false`, open to a task's `run_settings`, makes the PR-Review row mail the parent orch a `[pr-review request]`.
This setting could replace it for most runs.


## Issue comment, 2026-11-08

Constraint from the operator: a parent orch is often very busy with coordination (the LoomYard hub's orch especially), which is why the parent review was dropped before. The design should keep the parent's load small: the review is opt-in per task, so a busy orch's hub leaves it off; the orch can hand the review to a fork of itself, which keeps its knowledge of the run but keeps the review out of the orch's main thread; and a deadline lets the run go on to Plan without the parent's review, recording that it did.
