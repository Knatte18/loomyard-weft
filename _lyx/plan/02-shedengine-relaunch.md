# Card 2 — shedengine-relaunch

**Create:**
- `plan:internal/shedengine#CheckGotoTarget` -> `func CheckGotoTarget(producers []ProducerDef, st Status, target, runID string) error`
- `plan:internal/shedengine#RelaunchRequest` -> `type RelaunchRequest struct`
- `plan:internal/shedengine#Relaunch` -> `func Relaunch(req RelaunchRequest) (Status, error)`
- `plan:internal/shedengine#TestRelaunch` -> `func TestRelaunch(t *testing.T)`

**Edit:**
- `internal/shedengine#Goto`
- `internal/shedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the status-file mutation the driver's re-launch performs when it consumes an arbiter's `resume` verdict (D6), in `goto.go` beside `Goto`, since shedengine owns every status-file write.

`CheckGotoTarget` returns nil when `target` names a producer of `producers` and `gotoAdmitted` admits it for `st`, and otherwise the error `Goto` returns today for that case: the unknown-target refusal naming the producers, the awaiting refusal naming the admitted rows, or the past-the-current-row refusal.
`Goto` calls it in place of its own inline target checks, so its messages stay byte-identical and the existing goto tests keep passing unchanged.

`RelaunchRequest` carries `StatusPath`, `LockPath`, `StatusLockPath`, `Producers`, `RunID`, `MissingStatusWayForward`, `Target` (empty for no goto) and `Admit func(cur Status) error`.
`Relaunch`:

1. creates the lock directories and takes the run lock with `lock.TryAcquireWriteLock`; a held lock returns an error wrapping `ErrShedBusy` with `busyWayForward`, and `Admit` is never called;
2. in one `state.UpdateJSON` under one hold of the status lock: refuses a missing file, an invalid state, a `done` run ("relaunch never re-opens a finished run") and a `running` run (another loop owns it), each with a way forward; with a non-empty `Target` runs `CheckGotoTarget`; then calls `Admit(cur)` and returns its error unchanged with nothing written; then, with a `Target`, moves the run as `Goto` does (current producer, `paused`, cleared error and transient, a `goto` history entry, recomposed activity) except that `PauseRequested`, `PauseBefore` and `PauseAfter` are kept as they stand; without a `Target` it writes the status unchanged;
3. releases the run lock on return; it calls no `CommitStatus`, as `Goto` does not.

The checks run before `Admit` so a refused target never lets `Admit` write its side effect (the caller's applied mark).
Bound: `Relaunch` performs a goto that keeps a human's pause, which `Goto` clears; it moves only backward through `CheckGotoTarget`, only under the run lock, and only when the caller's `Admit` passes.

`TestRelaunch` is one table-driven test over a temp status file, reaching each refusal from a status file in that state after the lock is taken: a run lock held by the test (busy, `Admit` not called, file unchanged), `done`, `running`, a target past the current row (`Admit` not called), an `Admit` error (file unchanged, error returned verbatim), a valid backward target with `pause_before` and `pause_after` set (moved, paused, `goto` entry appended, both conditions kept), and no target (`Admit` called once, file content unchanged).
Raise `internal/shedengine`'s row in the test budget by one.
Describe `Relaunch` beside `Goto` in `doc.go`.

**ImpactSummary:** `Goto` keeps its behavior and messages; the new function has no caller until card 12.
