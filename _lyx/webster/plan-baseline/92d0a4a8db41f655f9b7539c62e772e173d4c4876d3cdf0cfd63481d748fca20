# Card 11 — board-runlock-names-cancel

**Edit:**
- `internal/boardengine#RunLockedError.Error`
- `internal/boardengine_test#TestRunLock`
- `internal/boardengine/doc.go`

**Intent:**
The board's run-lock refusal tells a caller abandoning a run to clear the status with `lyx board set-status`; with the cancel verb that way forward becomes `lyx batten cancel <slug>` (Scope: way-forward texts).
`RunLockedError.Error` keeps its first clause (record the finding as a note with `lyx board upsert`) and words the unlock as: run `lyx batten status <slug>` from the hub's prime worktree, and to abandon the run end it with `lyx batten cancel <slug>` from the prime, which ends its processes, removes its records and clears this status; it no longer names `lyx board set-status`.
The package doc's sentence on that message's ways forward (`doc.go`, under the run lock) is reworded to match: an abandoned run is ended with `lyx batten cancel <slug>` from the prime, which clears the status.
The grep for the old text found no test asserting it.
`TestRunLock` gains an assertion on its refused-write row that the error names `lyx batten cancel <slug>` and does not name `lyx board set-status`.

**ImpactSummary:** Message text only; callers match the refusal through `errors.Is(err, ErrRunLocked)`, which is unchanged.
