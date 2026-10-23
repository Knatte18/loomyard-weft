# Card 41 — batten-drops-pr-gate-lock-arms

**Delete:**
- `internal/battenshed#innerRunProducer.handedLockStep`
- `internal/battenshed#innerRunProducer.parentMovedUnderGate`
- `internal/battenshed#innerRunProducer.staleBaseResume`
- `internal/battenshed#failStaleBaseDecision`
- `internal/battenshed#failParentMoved`
- `internal/battenshed#handedRecord`
- `internal/battenshed#lockGateRow`
- `internal/battenshed#TestInnerRun_ParentLockKeepsOrReleasesAHandedRecord`
- `internal/battenshed#TestInnerRun_StaleBaseResumesAnAwaitingChildOnce`
- `internal/battencli#parentMovedSinceMergeIn`
- `internal/loomcli#battenWatcher`

**Edit:**
- `internal/battenshed#ChildParentLock`
- `internal/battenshed#lockWatch`
- `internal/battenshed#childWait`
- `internal/battenshed#innerRunProducer.lockStep`
- `internal/battenshed#innerRunProducer.awaitingStep`
- `internal/battenshed#innerRunProducer.wait`
- `internal/battenshed#innerRunProducer.noticeCondition`
- `internal/battenshed#newLockHarness`
- `internal/battenshed#failureLockSeams`
- `internal/battenshed#TestInnerRun_ReadFailuresLogOncePerEpisode`
- `internal/battenshed#TestInnerRun_OwnerCheck`
- `internal/battencli#childParentLock`
- `internal/battencli#TestChildParentLock`
- `internal/loomcli#loopLockExit` -> `func loopLockExit(locks *parentlock.Store, self func() parentlock.Owner) func(stop shedverbs.LoopStop, st shedengine.Status)`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#TestLoopLockExit`

**Extends:**
- `internal/battenshed/doc.go`

**Intent:** Remove batten's parent-lock backstop arms for a PR-Gate hand-off and a moved parent at PR-Gate, keeping its idle release (D13).
The loop's exit hook no longer hands a record to a watching batten at an awaiting PR-Gate: `loopLockExit` releases every record the loop owns at every stop, so `battenWatcher` goes.
With no hand-off, batten never owns a handed record: `lockStep` drops its handed-record branch and `handedLockStep` goes, and `ChildParentLock` drops `SelfPID`, `GateRow` and `ParentMoved`.
The stale-base resume of a child awaiting at PR-Gate goes with `parentMovedUnderGate` and `staleBaseResume`: `awaitingStep`, `wait` and `noticeCondition` stop calling them, `childWait` drops the bookkeeping fields, and battencli's moved-parent reader goes.
The idle release of a stalled loop's record (`loopLockStep`), the owner-pause arm and `ownedElsewhere` stay.
The historical classifier `NoticeRules.GateRow`, which reads old runs, is not touched here.

The handed-record and stale-base tests go; the lock harness, the read-failure and owner tests and battencli's lock-seam test drop the removed seams; `TestLoopLockExit` asserts a release at an awaiting stop with a watching batten.

`doc.go` drops the hand-off and stale-base paragraphs.

**ImpactSummary:** Two batten lock arms, the loop's hand-off and their seams go; their fillers are battencli's lock seams and loomcli's `specFor`, all listed.
