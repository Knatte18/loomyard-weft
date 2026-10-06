# Card 8 — step-error-keeps-handoff-voucher

**Edit:**
- `internal/loomcli/halt.go`
- `internal/loomcli/arm.go`
- `internal/loomcli/entryobservation.go`
- `internal/loomcli/halt_test.go`

**Intent:** D6: the entry observation never reports a crash-resume for a run whose previous step completed and was handed off cleanly; the reported case is last history entry `Webster / done`, state `running`, the next producer not yet started (#396, #392).

Cause, from the code: `Shed.persistTransient` writes the status file and then calls the `CommitStatus` seam; when that commit fails (the Webster row has just committed its own run record, and a concurrent `lyx board sync` or other git write holds the index), `shed.Step` returns the commit error after the `Webster / done` transition is already on disk.
The generic step body then skips `PostStep`, so no handoff voucher is recorded, `loomAfterStep`'s `failedHalt` finds the status `running` rather than `failed` and does nothing, and the next step's entry observation sees state `running`, a non-empty history, a free run lock and no voucher: a crash-resume.
A step process that returns at all did not crash, whatever its error.

Reproduce first, through the CLI step chain: in `TestStep_CrashResumeNoteFollowsTheHandoffVoucher`, add a row whose first step's shed carries a `CommitStatus` seam that fails on the `Row-A` done transition, so the first step returns a producer error with the transition persisted, and the second step must write no `loom-crash-resume` note.
Run it before the fix and see it write the note.
If it does not, this is not the cause: reproduce the reported sequence through the same `handoffSteps` chain with each remaining candidate (a history entry appended after `loomPostStep` recorded the voucher; a step path that bypasses `PostStep`; a second entry observation consuming the voucher), fix at whichever reproduces, and record the finding in the commit message; the bound below holds either way.

Fix: add `recordHandoffVoucherFromStatus` in `entryobservation.go`, which reads the status file strictly (as `failedHalt` does) and records the voucher with the persisted history length and state, warning and recording nothing when the read fails.
In `loomAfterStep`'s `stepErr` branch, unless the error is `shedengine.ErrShedBusy` (another driver owns the run, and its own exit vouches), call it before the `failedHalt` check.
The success path keeps `loomPostStep`'s voucher unchanged.
Update `recordHandoffVoucher`'s doc comment, `loomAfterStep`'s and `loomPostStep`'s in `arm.go`, which says the voucher is recorded after a successful `shed.Step`, to say that every step that returns, with or without an error, vouches for what it left on disk.
`arm.go` is also where the fallback below fixes a step path that bypasses `PostStep` or a voucher read in `loomPreStep`.

Bound: a driver that dies before it records the voucher, mid-producer or between a finished step and the voucher, never reaches `loomAfterStep` and is still reported; a busy refusal records nothing; a refusal from `PreStep` or `BuildShed` runs no step and records nothing.
The existing `MarkerRemoved` row, which deletes the voucher between two clean steps, keeps proving that a death right after a finished step is still noted.

**ImpactSummary:** loomAfterStep's error branch now records the handoff voucher from the status file; only the next entry observation reads it, so only false crash-resume notes after a returned step disappear.
