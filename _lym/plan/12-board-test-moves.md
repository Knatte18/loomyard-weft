# Card 12 — board-test-moves

**Create:**
- `plan:internal/boardengine_test#TestReconcileOrigin_EndedRuns` -> `func TestReconcileOrigin_EndedRuns(t *testing.T)`
- `plan:internal/boardengine_test#TestReconcileOrigin_OwnedElsewhere` -> `func TestReconcileOrigin_OwnedElsewhere(t *testing.T)`
- `plan:internal/boardengine_test#TestPush_BoardReplay` -> `func TestPush_BoardReplay(t *testing.T)`
- `plan:internal/boardengine_test#reconcileHub` -> `type reconcileHub struct`
- `plan:internal/boardengine_test#newReconcileHub` -> `func newReconcileHub(tb testing.TB) *reconcileHub`
- `plan:internal/boardengine_test#writeRunStatus` -> `func writeRunStatus(tb testing.TB, prime *lymcwd.Location, slug, producer string)`
- `plan:internal/boardengine_test#teardownRecorder` -> `type teardownRecorder struct`
- `plan:internal/boardengine_test#requireEntry` -> `func requireEntry(tb testing.TB, report fabriclayout.OriginReport, slug string) fabriclayout.EntryOutcome`
- `plan:internal/fabricengine#RunStatusFileForTest` -> `func RunStatusFileForTest(prime *lymcwd.Location, slug string) string`

**Delete:**
- `internal/fabricengine_test#TestReconcileOrigin_Ended`
- `internal/fabricengine_test#teardownRecorder`
- `internal/fabricengine_test#endedTemplate`
- `internal/fabricengine_test#arrangeEnded`
- `internal/fabricengine_test#endedHub`
- `internal/fabricengine_test#writePairStatus`
- `internal/fabricengine_test#publishBoard`
- `internal/fabricengine_test#originBoard`

**Edit:**
- `internal/fabricengine_test#TestReconcileOrigin_Moves`
- `internal/fabricengine_test#TestReconcileOrigin_Judge`
- `internal/fabricengine_test#TestPushFailureClasses`
- `internal/fabricengine_test#TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`
- `internal/fabricengine_test#TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks`
- `internal/fabricengine_test#movesPair`
- `internal/fabricengine_test#writeStatus`
- `internal/fabricengine_test#newFailureFixture`
- `internal/fabricengine_test#TestBoltReplayers`

**Extends:**
- `internal/boardengine/doc.go`

**Uses:**
- `internal/testkit/boardkit#InstallReplay`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPair`
- `internal/fabricengine#ReconcileOrigin`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#CoalescePushRequest`
- `internal/fabricengine/doc.go`

**Intent:**
D12's moves of this concern's tests: what rests on the board's replay moves to `internal/boardengine`, and this concern's fabric tests stop importing `internal/testkit/boardkit`, `internal/boardengine`, `internal/shedrun` and `internal/shedengine`.

A `ReconcileOrigin` or records push reads the board's run ledger only when a recorded run, an owner question or a Bolt replay needs it (`readOriginEnded`, `readOriginOwners`, `retryBoltPush`).
So a fabric test that records no run, asks no owner and diverges no Bolt branch needs no replayer, and the rows that need one pin the board module's ended set, owners or entry merge.

Moved, as tier1 tests in package `boardengine_test`, which imports `internal/hubforge` and `internal/testkit/boardkit` with no cycle, and which register the real replay through `boardkit.InstallReplay`:
- `TestReconcileOrigin_EndedRuns` takes `TestReconcileOrigin_Ended`'s rows that pin which runs origin's board ends:
  done entry with a local pair torn down once and reported landed; ended with no local pair; Finalize before the parent merge; past Finalize; the four Raddle rows; entry removed; local-only Bolt entry; pair with no run record; the run-scope rows; and the vanished branch's failed-push record cleared only by a moving reconcile.
  It writes run records through the installed `fabricengine` seams' paths by way of shed's writer, `shedrun.StatusFile` and `state.UpdateJSON`, which a test outside fabric may import.
  It keeps the earlier-snapshot-tag row, which pins the engine's `SnapshotTagged` read, and asserts the reconcile's exact printed `mutations` array, the teardown's entries included (D4, D5).
  The busy, failing-teardown and judge-only rows are deleted, held by card 10's `TestEndedTeardowns`.
- `TestReconcileOrigin_OwnedElsewhere` takes `TestReconcileOrigin_Moves`' owned_elsewhere rows: the summary and way forward per subject and scope, the records branch left unreplayed while fast-forwards move, and ended-and-owned-elsewhere; and `TestReconcileOrigin_Judge`'s entry-with-a-run-record and `PairMayBeAbsent` entry rows, and the Bolt rows (needs_replay of a diverged Bolt, the Bolt replay that sets aside an ended entry's status, the dirty Bolt that stops every entry, the both-sides prime weft records change).
  It keeps the nil-`OwnerOf` and `OwnerOf`-error rows, which pin `readOriginOwners`' early return and its wrapped seam error over a recorded run.
- `TestPush_BoardReplay` takes `TestPushFailureClasses`' `ABoltPushRejectedOverAnUnrelatedOriginEntryIsReplayedAndLands`, `ABoltPushRejectedWhileTheBoardLockIsHeldDefersWithADeferredRecord`, `APrimeWeftPushRejectedAfterThisMachinesFinalizeKeepsTheLandingsCommits`, `ADetachedWeftPushRejectedAsNotFastForwardIsReplayedAndPushed` and `AParkedPushOfABranchGoneOnOriginDoesNotRecreateIt`;
  `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`'s `ALandingCommitPastOriginHoldsTheRecordsUntilTheRunIsDone`;
  and `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks`' two `OwedUnrecorded_*OfAnEndedEntry*` rows.
  Each keeps its tier and drops every bare side spelling the vocabulary scan refuses outside fabric's owner set, using `hubforge` accessors in place of side-named helpers.

Rewritten in place:
- `TestReconcileOrigin_Moves` and `TestReconcileOrigin_Judge` drop `boardkit.InstallReplay`, `boardengine.RunStatus`, `shedrun.StatusFile` and `shedrun.PrimeRunLock`;
  `movesPair`'s template writes no run status, so the kept rows record no run and read no ledger.
  Each kept row of cards 9 and 11 that asserted its entry (the clean and dirty pair weft, the live pair, the held weft lock, the held Bolt push lock) asserts its branch outcome instead, state and way forward,
  since the entry's classification over those states is `TestClassifyEntries`' rows.
- `TestBoltReplayers`' row "With no replayer registered the ended-set read and the owner read fail with ErrReplayerMissing" writes its own run record through `writeStatus` before its reconcile, so it still reaches both ledger reads.
- `writeStatus` takes the status path from the installed run-records seam through `RunStatusFileForTest` (`export_test.go`) in place of `shedrun.StatusFile`; `newFailureFixture` drops `boardkit.InstallReplay`.
- The moved tests build on fixtures of their own in `boardengine_test`, which cannot see the `fabricengine_test` helpers:
  `reconcileHub` and `newReconcileHub`, a hubforge pair with the replay installed and the prime's records published;
  `writeRunStatus`, a run record written through `state.UpdateJSON` over shed's `Status` at `shedrun.StatusFile`; `teardownRecorder`; and `requireEntry`.
  The fabric helpers whose last users move go: `endedTemplate`, `arrangeEnded`, `endedHub`, `writePairStatus`, `publishBoard` and `originBoard`.
- `TestPushFailureClasses`, `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` and `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` drop `boardkit`; a kept row that still reads the ledger is a defect of this card's split and moves with the rows above.
- `teardownRecorder` goes with `TestReconcileOrigin_Ended`, its last user in fabric, and the moved tests carry their own recorder.

The commit message lists every deleted test and row and, for each moved or kept test, the rule it alone guards (overview, Shared Decisions).
The boardengine doc addition says the package's tests hold fabric's flows that rest on the board's replay.

**ImpactSummary:** Test-only: three tests join internal/boardengine and this concern's fabric test files lose their board and shed imports; no production code changes.
