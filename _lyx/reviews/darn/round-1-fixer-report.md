# Fixer-report, darn round 1

## Changes

All four findings fixed as proposed, in `internal/shedverbs`; one commit for F1–F4 (their edits share files and hunks) and one follow-up commit.

- F1: `LoopPIDRecord.inFlightAt(status)` (`loopwait.go`) holds the record's child as in flight only while the status file reads the recorded producer and history length.
  `deadLoopStop` reads the status before the stop and leaves out "step X was in flight" and the trace id when it has moved.
  `loopExitStop` takes the trace file from the record's trace only when `inFlightAt(before)`, otherwise the loop log.
  `doc.go` carries the qualifier.
  The test table in `loopwait_test.go` now sets the status to the record's producer and history length for the in-flight rows, and has a new moved-status row asserting no clause and the loop log.
- F2: `TestLoopIntegration_ChildTreeDiesWithLoop` asserts the dead-loop `loop.detail` holds the killed-step-tree clause and the in-flight step with its trace.
- F3: the early-exit subtest sets `AfterInterrupt` and asserts the cause `loop-exited: loop <id> exited before it recorded itself; its output is in <log>` and the loop log as trace file.
- F4: the helper child writes the trace id it runs under to `trace.id`, and the test compares the pid record's `TraceID` with it.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: `StatusMoved` duplicated the `inFlightAt` comparison — fixed as proposed (`rec.hasChild() && !rec.inFlightAt(after)`).
- fix-consistency, LOW: `doc.go` qualifier ambiguous between producer and trace id — fixed as proposed.

## Disputed

None.

## Deferred

None.
