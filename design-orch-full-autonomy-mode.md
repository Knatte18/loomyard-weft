# Autonomy mode: the parent orch settles every halt, operator only for severe halts and the beam merge

`orch-full-autonomy-mode` · Task · sketch, orch, drift, landing, dobby, batten, pawl, config · starting · Seed-IDE

Decision record: [Sketch](decision-orch-full-autonomy-mode.md)

## Goal (operator, 2026-11-22)

An autonomy mode under which the run's parent orch receives every halt of every member run and settles it itself.
Whether a beam is autonomous is decided per beam: one beam may run autonomous while another beside it does not.
The operator is reached only in two cases:

1. A severe halt the orch cannot settle.
2. The beam is ready to merge: lym opens the beam's pull request ready for merge, then notifies the operator.

Wanted first in the Models hub on OSL-1046 and in loomyard.

## What was seen

Runs halt often for the operator: one halt per drift evaluation, one per pull request of a member task, and other parent-only stops.
Drift blocks are the largest share in recent runs.

## What exists

- Run agents put their questions to the parent orch (e55618bb).
- `code_review_ask_parent` and `judge_ask_parent` let the parent see and veto code review rounds and drift judge rulings (8998a865); a drift does-not-match ruling still stops the run.
- Halts reach the parent as batten and dobby notices, but the run then waits for a resume, a decision or `pr-continue` that the operator often gives.

## Open for the design

- Which halts the orch may settle, and how a halt is marked severe enough for the operator.
- How member PR review is handled under the mode: off, or reviewed by the parent.
- The setting: a per-beam switch, and whether a hub or the prime's root tasks can take it too; it sets the existing ask keys and the new halt routing.
