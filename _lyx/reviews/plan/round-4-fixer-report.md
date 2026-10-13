# Round 4 fixer-report — plan

## Changes

- **F1** (fixed as proposed).
  Premise checked against `innerrun.go` (`wait`, `check`, `awaitingStep`'s hard end on a `ReadDecision` error), `innerrun_parentlock.go` (`lockStep` returning on a covered pause before the read, `handedLockStep` → `resumePending`, `parentMovedUnderGate`), `innerrun_pause.go` (`runArm` skipping the arm under a pause) and `notice.go` (`noticeStep` → `stateChangeReady` → `undecided`).
  In `09-battenshed-read-guards.md`: removed the `innerRunProducer.handedLockStep` Edit target, the handed-lock decision read from Intent's guarded sites and its guard row;
  added the Intent sentence that the stale-base and notice-step decision reads repeat only while a pause covers the slug;
  added that those two rows run with `InnerRunDeps.Pause` covering the slug, their shared `ReadDecision` seam failing on its first six calls, and the per-row WARN, DEBUG and recovery INFO assertions.
  In `00-overview.md`: took `handedLockStep` out of the battenshed guard row and added its keep row.
- **F2** (fixed as proposed).
  Premise checked against `innerrun_halted_test.go`'s test, `newInnerRunDeps` (`NoticeProbe: time.Hour`, `pauseAtSleep = 1`) and `haltedStep` calling `reviveDeadDriver` only on `w.probeDue`.
  The card's sentence now sets `deps.NoticeProbe = 0` and `clock.pauseAtSleep = 4` and has `DriverStrand` answer `ChildDriverLive` after its three failures, so the `ReviveStrands` stub is never called.

## Disputed

None.

## Deferred

None.
