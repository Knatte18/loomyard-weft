# Plan-Bouncer and Batchifier pass a plan that webster begin-batch then refuses with delete-before-reference

`plan-drift-refusal-route` · Note · bug, webster, loom

Imported from [issue #625](https://github.com/Knatte18/loomyard/issues/625).

## What happened

A task's run passed Plan-Bouncer (done) and Batchifier (done), then Webster's `lyx webster begin-batch 17` refused with `plan_drifted`.
The blocking finding was `delete-before-reference/18-retired-builders-delete[blocking]`: a card deletes a symbol that a later card still references in its Edit code.
`begin-batch 17` was refused twice with the identical finding, and the run halted `blocked` at Webster with anomaly `escalation-to-human`.

## Why it is abnormal

- The `delete-before-reference` check is a pure plan-ordering check, yet it only fires at `begin-batch`, after the whole Discussion and Plan review pipeline and the Batchifier have accepted the plan.
  Plan-Bouncer and Plan-Burler should have caught it, or the Batchifier should have refused to batch the plan.
- The refusal is labelled `plan_drifted`, but the plan did not drift from the tree between planning and batching; it was malformed from the start.
  The label points an operator at the wrong cause.
- The refusal text already names the way forward (move the delete to a later card, `lyx webster rebaseline --card NN`, then `begin-batch` again), but nothing routes that back to a plan-editing agent.
  The run escalated to the human after roughly 45 minutes of Webster time.

## What to do

- Run the `delete-before-reference` finding (or the whole plan re-resolution) at Plan-Bouncer or Batchifier time, so an ordering defect is fixed by the plan reviewers.
- Distinguish a plan malformed at authoring from true drift in the refusal's classification.
- Consider letting Webster or the driver route a `delete-before-reference` refusal to a plan-fix step instead of halting for a human.

Provenance: task `overview-seats`; notes `loom-halt.md`, `webster-refusal-begin-batch.md`, `webster-refusal-begin-batch-2.md`.

## Correction (orch, 2026-10-15)

In `overview-seats` the plan was not malformed: the reference was a temporary parity test that card 12's batch added after Plan-Write, inside a region card 18 itself rewrites, so the finding was a false positive of the check, fixed by `delete-order-own-edit` (landed 2026-10-14).
What stays valid here: the `plan_drifted` label names the wrong cause, and a real `delete-before-reference` refusal at `begin-batch` has no route back to a plan-editing agent and halts for a human.

