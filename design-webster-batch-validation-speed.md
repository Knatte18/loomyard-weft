# Webster begin-batch spends a minute validating the plan per batch

`webster-batch-validation-speed` · Notes · bug

Seen on `test-suite-change-amplification` (2026-10-03), a 45-card plan that the Batchifier split into one card per batch.

## Measured

- `lyx webster validate` on that plan takes 64 s of wall time at 100 % of one core, with no child processes: the time is spent in-process, single-threaded.
- `begin-batch` runs `planglyph.ValidateDispatch` over the whole plan before every batch (`internal/websterengine/beginbatch.go:303`), so each batch pays roughly that cost again.
- The gap from one batch's report to the next batch's spawn was 143 s, 72 s and 81 s for the first three batches, while the forks themselves took 1–2 min each.
  Over 45 batches that is on the order of 50 minutes of setup.

## Cause

Not loomyard's own code: `quarry.Repo.Resolve` re-parses every file in a target's directory for each file target (Knatte18/quarry#34).

## Per-batch validation stays (operator)

Validating before every batch is deliberate.
A fork may drift from the plan when it finds something the plan missed, and the plan changes with it; the per-batch check is what notices.
It is also the base for running forks in parallel later: glyphs, quarry and the validator maintain, at every batch, a DAG that decides which cards and batches can run concurrently.

## Want

- quarry#34 fixed and the quarry dependency bumped, so one validation of a plan this size takes seconds.
- If that is not enough, make the per-batch validation incremental: re-resolve only the targets in files the last batch's delta touched, plus the cards that reference them, and reuse the rest from the previous batch.
- Drop `begin-batch`'s per-batch model assertion (operator): the fork inherits Merriam's model, so `begin-batch` has no model to check.

## Related

The one-card-per-batch split makes the per-batch cost dominate; whether the Batchifier should group cards more is a separate question.


From GitHub #345 (intake 2026-10-04).
