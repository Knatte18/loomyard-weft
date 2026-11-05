# Fixer-report, round 1 (code)

Gate: `lym gate test ./internal/shedverbs` tier 0 passes; tier 1 passed before the wording-only follow-up commit, including the two new and changed integration tests.
The fixes went into two commits, not one per finding: F2, F7, F8, F9 and F10 share `loop.go`, so they could not be split cleanly.
After the gate, the wrapped comment at `doc.go:111-112`, one sentence split at a list comma, became two sentences of one line each; no other change.
F3 is a `decision`-class finding and went to the parent; it is not in this report.

## Changed

- **F1**: `deadLoopStop` appends the step stderr clause only while `rec.inFlightAt(status)`, and `loopExitStop` fills `loop.stderr_path` and `loop.trace_copy` only while `rec.inFlightAt(after)`.
  The StatusMoved row now has trace files and a stderr file, and asserts no stderr clause, an empty `stderr_path` and an empty `trace_copy`.
  `doc.go` and the two function comments say so.
- **F2**: narrowed the claims, the reviewer's second option.
  `doc.go` and the `runLoop` comment name a panic outside the stepping and the early returns as further ways to end without an envelope.
- **F4**: the TraceKnown row writes real trace files and asserts `loop.trace_copy` equals the `StopFiles` path and holds the files joined in order.
  The other rows assert it empty.
- **F5**: new row `NoStartingMark_OnlyTheLoopExitClause`.
- **F7**: `recordTerminatingSignals` returns a second function that reports whether a signal arrived, after waiting for `Raise` to return.
  `runLoop` checks it after the stepping and before the envelope write, and returns 1 without a failed stop, interrupt hook or envelope.
  `TestLoop_TerminatingSignalIsMarkedInTheLoopLog` now asserts no envelope and the pid file left.
- **F8**: `raiseSignal` waits `raiseGrace` after sending the signal, then exits with 128 plus the signal number.
- **F9**: `appendLoopLogMark` and `newlineIfUnterminated` open every mark on its own line; the start, exit and signal marks use them.
  `TestLoopIntegration_SpawnMarksTheLoopsExit` now ends its output without a newline.
- **F10**: new `TestLoopIntegration_PanicKillsTheInFlightStepTree` panics through the loop's clock with a real process group in flight and waits for it to die.
  The stack claim is dropped from `doc.go` and the `stepUntilStop` comment, the reviewer's other option for that half.
- **F11**: an old-1 mark now sits inside the dead-1 span in the `LoopExitMarksAndStderr_OnlyTheDeadLoopsLines` row.

## Disputed

- **F6**, ground `premise false`: `runChild` calls `record(proc.TreeRecord{})` when the child exits (`loop.go`, after `logger.Info("shed: loop child exited"...)`), and the callback in `stepUntilStop` assigns that zero record to `pid.Child`.
  Between steps `pid.Child` is therefore zero, and `killStepTree` returns at its `PGID == 0` guard (`loopenvelope.go`), so the recovery never reaches a finished step's group.

## Deferred

Nothing deferred by me.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: `OnExit` still documented as running at every stop. Fixed in `LoopSpec.OnExit`, the `runLoop` comment and `doc.go`.
- fix-consistency, LOW: `LoopSpec.Raise` comment omitted the exit after the grace. Fixed.
- fix-consistency, LOW: the early-return list in `doc.go` read as closed. Fixed by wording it as examples ("such as").
- fix-proposal, LOW: `doc.go` stated the signal path's skips as absolute. Fixed with "once the loop has seen the signal".
- fix-proposal, LOW: a custom `Raise` that never returns would hang `runLoop` at the signal check. Disputed, `premise false`: `Raise`'s contract is to end the process, so a `Raise` that does not return is the contract, and the production `raiseSignal` always ends the process.
