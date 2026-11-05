---
round: 2
focus:
  - "internal/shedverbs/loopwait.go, deadLoopStop and loopExitStop: when the status file has moved past the child recorded in the pid file (loop dies after Drift-Await done, before the Webster start), do the step stderr clause, loop.stderr_path and loop.trace_copy still come from the finished step? Settle whether they are gated on the same in-flight condition as the in-flight clause."
  - "internal/shedverbs/loop.go, the panic recovery in stepUntilStop and the claim in doc.go that a loop ends without an envelope only by a signal or a machine failure: does a panic in OnStart, AfterInterrupt, OnExit, WriteFailedStop or the code after the stepping still end the loop with no envelope? Settle whether code and claims agree."
  - "internal/shedverbs/loop.go, recordTerminatingSignals and raiseSignal: after a terminating signal, can the main goroutine still write a failed stop or run AfterInterrupt before the process ends, and does an inherited-ignored SIGHUP or SIGINT leave the loop alive after the mark says it ends without an envelope?"
  - "internal/shedverbs/loopwait.go, appendLoopLog and the start mark: when the loop's last output line has no trailing newline, does the appended mark still open its own line so loopExitEvidence finds it?"
---

Where the code changed after round 1, verify the new text against the same questions and against the tests that now claim to pin it.
