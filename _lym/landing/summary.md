# shedverbs: record how a loop ended in the dead-loop halt note

A loop that ends without an envelope now leaves evidence of how it ended, and the halt note carries it.
Before, the `loop-exited` stop said only that no envelope was written, so an operator could not tell a loop crash from an external kill.

Closes #51

## What changes

- `spawnLoop` (`loopwait.go`) marks in the loop log that a loop is starting, before its process starts, and the process's pid and exit status once the spawn reaps it.
  Marks open with `lym shed: loop <loop id> `, which tells them from the output of the loop and its children, and each opens its own line even after output with no trailing newline.
- A loop that receives SIGINT, SIGTERM or SIGHUP (`recordTerminatingSignals`, `loop.go`) writes a mark to the loop log and then ends through `Spec.Loop.Raise`.
  By default `Raise` restores the signal's default action and sends it to the process again, so the process ends by the signal it was sent.
- A panic while stepping (`stepUntilStop`, `loop.go`) is recovered: the in-flight child's step tree is killed, and the loop ends in an interrupted stop with the cause `loop-panicked: <value>`.
  It writes its envelope as for any other interrupted stop.
- The dead-loop stop and the early-exit stop (`deadLoopStop`, `loopExitedEarlyStop`) add to `loop.detail`:
  - how the loop ended, read from the loop log after the loop's last starting mark: the loop's own marks, or `loop exit: not recorded` when there are none, then an excerpt of the loop's other lines (a Go crash's first lines, else the last lines);
  - while the status file has not moved past the in-flight child, an excerpt of that child's captured stderr.
- Under the same condition the stop's envelope fills `loop.stderr_path` and `loop.trace_copy`, which the way-forward text already names but a dead loop never filled.
- `stderrExcerpt` is split so the same excerpt logic serves log lines (`outputExcerpt`).
- `doc.go` and the doc comments describe the new marks, the signal and panic paths, and the detail layout.

## Design decisions

- A terminating signal does not make the loop write an envelope, and the issue's ask to make a no-envelope exit write one is met only for a panic.
  SIGTERM reaches the loop from `runproc.End` on a cancel, a teardown or a reap.
  A failed stop written inside the dying loop's kill grace would file halt notes for deliberate cancels and race the teardown's mark.
  The goal, telling a crash from an external kill, is met by the recorded signal mark that the next waiter's dead-loop stop reports.
- After a signal is seen the loop runs no `OnExit` hook and writes no failed stop.
  `OnExit` is documented as running at every stop but a terminating signal's while the loop steps.
- Panics outside the stepping, in a hook or in the code after it, are not recovered and still end the loop without an envelope.
  Their evidence is the exit mark and the Go crash lines in the loop log.
- `Spec.Loop.Signals` and `Spec.Loop.Raise` are seams so tests drive the signal path without signalling the test process.

## To check by hand

- Not run: no real loop was killed by a signal end to end on a live run.
  `TestLoopIntegration_ChildTreeDiesWithLoop` now terminates the loop with SIGTERM and asserts the signal mark in `loop.detail`; run the integration tier on Unix to confirm.
- No test covers a loop log whose last starting mark for the loop is missing in the live spawn path.
  `loopExitEvidence` (`loopwait.go`) returns only the first clause then, and only the unit case `NoStartingMark_OnlyTheLoopExitClause` covers it.
- Panic recovery in `stepUntilStop` is narrower than the issue's "make that path write one" suggests: it covers only the stepping, as the first design decision says, and a panic in a hook or after the stop is final still leaves no envelope.
  Check that this scope is acceptable.
- `deadLoopStop` fills `loop.stderr_path` and `loop.trace_copy` only while the status file has not moved past the in-flight child.
  A dead loop whose status file has moved on shows neither, and the stderr of the last finished step is not offered.
  Check that this is the intended reading of "stale".
- `loopExitStop` writes `loop.trace_copy` through `copyTraceFiles`; `TestUntilStop_DeadLoopStopsLoopExited` checks the copy's contents, but no test covers a copy that fails to write.
- The choice of signal handling over an envelope write is recorded as a decision in this description only; it has no entry in the repository's design docs beyond the `doc.go` text.
