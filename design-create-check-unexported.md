# webster: create-not-done check fails on unexported plan targets, killing batches

`create-check-unexported` · Note · bug

Imported from [issue #511](https://github.com/Knatte18/loomyard/issues/511).

## What happened

Two batches in one loom run were refused by `lyx webster record-batch` with `create-not-done`, and the run halted `blocked` at `Webster` with anomaly `escalation-to-human`.

- Batch `01-fixture-git-marker`: `create-not-done/2-testtiming-resources[blocking]: Create target "plan:cmd/testtiming#rusageStats" still does not resolve`. `lyx webster recover-batch` then ended with status `dead`, `dead_reason` `asking`.
- Batch `08-reedcli-probes-onto-tmuxkit`: `create-not-done/10-tmuxkit-socket-check[blocking]: Create target "plan:internal/testkit/tmuxkit#tmuxAllowed" still does not resolve`. `lyx webster recover-batch 08` refused with the same `batch_failed` message.

Both unresolved targets are unexported (lowercase) identifiers.
That is the common factor visible in the notes; whether the target was genuinely never created could not be established from them.

## Why it is abnormal

- The failure repeats after recovery for the same target, so recovery does not change the outcome and the batch ends up dead.
- If the check cannot resolve unexported identifiers (or identifiers declared in test files), a correct implementation is still reported as not done, and the loom escalates to a human for no real defect.
- The refusal text does not say how the target was looked up, so the operator cannot tell a missing symbol from a resolver limitation.
- After `lyx loom goto` back into `Webster`, the loop halted again at once with "Batch 01-fixture-git-marker is dead: its recovery already failed terminally, so no later batch can begin", so a dead batch also blocks the way forward the halt tells the operator to take. The wording of that halt changed between runs ("was begun" vs "can begin").

## What to do

1. Check how `plan:<pkg>#<name>` Create targets are resolved and confirm unexported identifiers resolve.
2. Make the `create-not-done` message say which lookup was done and in which files.
3. Decide whether `goto` into `Webster` should be able to revive a dead batch, or whether the halt should say the operator must replace it.

Trace files are under `.lyx/logs/`; archived webster reports are under `_lyx/webster/reports/`.

Provenance: task `test-suite-weight`; notes `loom-halt.md`, `loom-halt-2.md`, `loom-halt-3.md`, `loom-halt-4.md`, `webster-refusal-record-batch.md`, `webster-refusal-record-batch-2.md`, `webster-refusal-recover-batch.md`, `binary-change-restep-20261010-121015.md`.
