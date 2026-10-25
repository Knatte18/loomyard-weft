# Card 13 — innerrun-short-check

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#innerRunProducer`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#innerRunProducer.runningStep`
- `internal/battenshed#innerRunProducer.Call`
- `internal/battenshed#innerRunProducer.check`
- `internal/battenshed#childWait`
- `internal/battenshed#innerRunProducer.foldOwnerTrigger`
- `internal/battenshed#innerRunProducer.judgeAgents`
- `internal/battenshed#innerRunProducer.lockStep`
- `internal/battenshed#innerRunProducer.loopLockStep`
- `internal/battenshed#innerRunProducer.reviveDeadDriver`
- `internal/battenshed#innerRunProducer.pauseRequested`
- `internal/battenshed#innerRunProducer.coverage`
- `internal/battenshed#runMarkerSuffixes`
- `internal/battenshed#newInnerRunDeps`
- `internal/battenshed#newRunShed`
- `internal/battenshed#TestInnerRun_WaitEndsOnlyOnAnEvent`
- `internal/battenshed#TestInnerRun_VerdictTable`
- `internal/battenshed#TestInnerRun_EachStateChangeIsItsOwnHistoryEntry`
- `internal/battenshed#TestInnerRun_DoneWaitsForLiveDriverThenFinishes`
- `internal/battenshed#TestInnerRun_DoneWaitIsNeitherCountedNorBlockedByTheRunningBudget`
- `internal/battenshed#TestInnerRun_HaltedStatesWaitExemptWithoutSpawning`
- `internal/battenshed#TestInnerRun_HaltedRevivesADeadDriverOncePerEpisode`
- `internal/battenshed#TestInnerRun_OwnerCheck`
- `internal/battenshed#TestInnerRun_ParentLockReleasesAStalledLoopsRecord`
- `internal/battenshed#TestInnerRun_RecordsWhetherItHoldsTheWatchedMarker`
- `internal/battenshed#TestInnerRun_ReadFailuresLogOncePerEpisode`
- `internal/battenshed#TestNotice_RestartDoesNotRenotify`
- `internal/battenshed#TestClearRunMarkers_RemovesEveryRunMarkerSuffixOnly`
- `internal/battenshed#fakeClock.Sleep`
- `internal/battenshed#newRestepHarness`
- `internal/battenshed#newLockHarness`
- `internal/battenshed#newNoticeHarness`
- `internal/battenshed#TestInnerRun_AwaitingWithoutApprovalWaitsExempt`
- `internal/battenshed#TestInnerRun_AwaitingRevivesADeadDriverAndKeepsItsHints`
- `internal/battenshed#TestInnerRun_AwaitingResumesOncePerApproval`
- `internal/battenshed#TestInnerRun_AwaitingAgainAfterResumeGetsReapproveHint`
- `internal/battenshed#TestInnerRun_AwaitingOldLayoutMarkerGetsReapproveHint`
- `internal/battenshed#TestInnerRun_AwaitingResumesOnARejectionOnceAndAgainOnASameSecondApproval`
- `internal/battenshed#TestInnerRun_AwaitingSpawnErrorWritesNoMarkerAndRetries`
- `internal/battenshed#TestInnerRun_AwaitingRefusedResumeRetriesWithoutMarkerThenResumes`
- `internal/battenshed#TestInnerRun_DoneLivenessErrorIsTreatedAsLive`
- `internal/battenshed#TestInnerRun_DoneFinishesPastGraceWithLiveDriverAndWarns`
- `internal/battenshed#TestInnerRun_UnparseableDoneSeenMarkerRestartsTheGrace`
- `internal/battenshed#TestInnerRun_StaleDoneSeenMarkerIsClearedWhileRunning`
- `internal/battenshed#TestInnerRun_HaltWarnsOncePerEpisode`
- `internal/battenshed#TestInnerRun_ReHaltAfterObservedResumeWarnsAgainAtTheSameHistoryLength`
- `internal/battenshed#TestInnerRun_PausedThenRunningThenDoneReachesDone`
- `internal/battenshed#TestInnerRun_StrandReadErrorSkipsTheReviveWithAWarn`
- `internal/battenshed#TestInnerRun_ARunningChildEndsTheReviveEpisode`
- `internal/battenshed#TestInnerRun_BinaryChangeRestepsADriverlessGoChild`
- `internal/battenshed#TestInnerRun_BinaryChangeSkipsALandingHalt`
- `internal/battenshed#TestInnerRun_SpawnsUntilASpawnIsConfirmed`
- `internal/battenshed#TestInnerRun_AdoptsARunningChildWithADriver`
- `internal/battenshed#TestInnerRun_FailedSpawnIsRetriedOnTheNextCall`
- `internal/battenshed#TestInnerRun_RunningArmReviewWaitNote`
- `internal/battenshed#TestInnerRun_NilSeamsDefaultToStdlib`
- `internal/battenshed#TestInnerRun_CancelledContext`
- `internal/battenshed#testInnerRunRefusedResume`
- `internal/battenshed#TestNewInnerRun_FloorsPollInterval`
- `internal/battenshed#TestStuckReport_LevelFollowsCause`
- `internal/battenshed#noticeHarness.producer`
- `internal/battenshed#TestNotice_StateChangeNotifiesOncePerEpisode`
- `internal/battenshed#TestNotice_StateChangeStartsNewEpisode`
- `internal/battenshed#TestNotice_ReturnToRunningStartsNewEpisode`
- `internal/battenshed#TestNotice_QuietOnlyForRunningChildWithLiveDriver`
- `internal/battenshed#TestNotice_NilNotifyRunsNoStep`
- `internal/battenshed#TestNotice_ParkedStopWaitsForTheStopReport`
- `internal/battenshed#TestNotice_SameStateRewriteMovesNeitherSinceNorTheKey`
- `internal/battenshed#TestNotice_DoneChildSendsNoNotice`
- `internal/battenshed#TestNotice_AwaitingLeavesTheRelayToTheDriver`
- `internal/battenshed#TestNotice_LiveArbiterHoldsTheStateChange`
- `internal/battenshed#TestNotice_DeliveryIsRecordedOnlyWhenQueuedAndRetriedWithinItsCaps`
- `internal/battenshed#TestNotice_LogsEachRunsSessionStateAtAQuietOrAPIErrorFinding`
- `internal/battenshed#TestNotice_EligibleStopHold`
- `internal/battenshed#battenshedAllowedImports`
- `internal/testkit/envkit#nilLegal`
- `internal/battencli#wireForHub`
- `internal/battencli#stepStepDrivenRunShed_ReturnsOnTheChildsStateChange`
- `internal/battencli#stepRunShedPausedChild_WaitsThenTearsDownOnceDone`
- `internal/battencli#stepAwaitingChildDoneTeardown_ArchivesTheRunRecords`
- `internal/battenshed/doc.go`
- `pattern/PATTERN-shed-driver.md`
- `docs/overview.md`
- `//contracts/recipes/batten-recipe.yaml`

**Delete:**
- `internal/battenshed#innerRunProducer.wait`

**Create:**
- `plan:internal/battenshed#innerRunMemory` -> `type innerRunMemory struct`
- `plan:internal/battenshed#innerRunMemoryFile` -> `func innerRunMemoryFile(scratchDir, producer string) string`
- `plan:internal/battenshed#innerRunProducer.loadMemory` -> `func (p *innerRunProducer) loadMemory() error`
- `plan:internal/battenshed#innerRunProducer.saveMemory` -> `func (p *innerRunProducer) saveMemory()`
- `plan:internal/battenshed#innerRunProducer.once` -> `func (p *innerRunProducer) once(ctx context.Context, w *childWait, step armStep) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/battenshed#TestInnerRun_MemoryOutlivesTheProcess` -> `func TestInnerRun_MemoryOutlivesTheProcess(t *testing.T)`

**Uses:**
- `plan:internal/shedengine#Yield`
- `plan:internal/parentlock#IdleWatch.Waiting`
- `plan:internal/parentlock#RestoreIdleWatch`
- `plan:internal/logger#EpisodeSet.Ticks`
- `plan:internal/logger#EpisodeSet.Restore`

**Intent:**
Turn the Run-Shed row's in-call wait into one check per Call, with its memory across checks in a per-run record, so `lyx batten run` becomes one short pass that dobby schedules (D4).
`once` replaces `wait`: it judges the probe due from the persisted `lastProbe` (due when `now - lastProbe >= noticeProbe`, or when none is recorded), and runs `check`'s full body once, never `wait`'s first round: `coverage`, `foldOwnerTrigger`, `checkChild`, `ownerStep`, `refreshWatched` and `lockStep` on a due probe, `noticeStep`, the arm's step and the pause-flag read.
A check that ends nothing returns `shedengine.Yield` with an empty pointer, so shed persists nothing and the pass ends; an end the check gives (a state change, the arm's event, a pause request, a failed stat, a hard error) returns as today.
`Call` keeps its read-before-spawn and spawn path, then calls `once`.
`Call` takes `childWait.begun` from the memory record's persisted episode state as `loadMemory` loaded it, before `observeState` runs, since `observeState` replaces the episode with the state just read; when `status.State` differs from that `begun`, `Call` ends at once with the state-change Stuck (`stateChangeText(begun, status)`) before any arm runs, so a state change that happened between passes ends the Call with its own Stuck and history entry even when `checkChild` would skip the unchanged file; with no persisted episode, `begun` is the state read, as today.
`innerRunMemory`, a tolerant JSON record at `innerRunMemoryFile` under the row's scratch directory written through `internal/state`, holds the inventory D4 names: the notice episode (state, since, `arbiterOffAt`, each delivery's attempts, last try, strand-warned and dropped), `driverBusyNoticed`, the owner trigger's `ownedBy`, `ownerNoticed`, `ownerDue` and `ownerCovered`, the `failures` episode ticks (`EpisodeSet.Ticks`/`Restore`), `restepWatch`, and from `childWait` the parent lock's idle readings and owner (`IdleWatch.Waiting`/`RestoreIdleWatch`), `resumeRetryAt`, `notParkedLogged`, `busyLogged`, `busyReason`, `busyKey`, `lastProbe`, `reason`, `reasonKeys`, `pauseWarned`, `coverageWarned` and the revive's start-fallback mark.
`loadMemory` runs at the start of `Call` and a missing or undecodable record reads as empty with one Warn; `saveMemory` runs on every return of `Call` and a failed write warns and changes no verdict.
`runMarkerSuffixes` gains the record's suffix, so a fresh seed clears it with the other run markers.
The owner trigger's own sleep watch (`ownerSleep`) and the idle checks' sleep watch go: `InnerRunDeps` gains `Sleeps func() ([]pauserecord.Interval, error)`, the sleeps the scheduling dobby recorded, which `judgeAgents` and the idle clocks read as excluded time beside the paused intervals (a nil `Sleeps`, or an error warned once per episode, excludes none), and `Awake` is the clock the caller keeps across passes; a wake reaches the owner verdict through the full pass dobby owes, so `foldOwnerTrigger` keeps only its pause-lift and owned-elsewhere-retry triggers.
`reviveDeadDriver` revives once per halt episode, read from the revived marker's presence rather than its pid, since each pass is a new process; the spawn confirmation stays per pass on purpose, so each pass re-checks the child's strand and locks and respawns a driverless running child.
Bound: a file that ends a wait between passes, such as a stop report or a strand's end, is seen at the next pass, up to one `notice_probe_s` later than today, and none is lost.
A yield is logged at Debug (`PATTERN-shed-driver`'s constraint for this task).
Under a covering pause the arm step still does not run, so a done child yields rather than returning Done, and no run enters Land under a pause (D12).
With `wait` gone, the spawn hold (`holdSpawnWhileCovered`) is the one reader of `InnerRunDeps.Sleep` and of the producer's `pollInterval` left, and card 14 deletes both when it turns the hold into a yield.
`runningStep`'s two reason texts stop naming a cadence, since a pass checks once: `inner shed run still running; dobby re-checks it on each probe or child status write`, and `inner shed run waiting: <review note>; dobby re-checks it on each probe or child status write`.
`envkit`'s `nilLegal` seam inventory gains `InnerRun.Sleeps`, which a nil leaves excluding no sleep, so `TestFullEnv_FillsEverySeam` keeps passing.
`battenshedAllowedImports`, the package's closed import allowlist, gains `internal/state`, which the memory record is written through and which battenshed's production code did not import before.
`doc.go`'s InnerRun section, the recipe file's header comment, `pattern/PATTERN-shed-driver.md`'s "Outside it" paragraph (which says batten's InnerRun wait polls a child run) and `docs/overview.md`'s batten sentences on the Run-Shed wait ("waits on its child inside its own call", "waited on without a time limit") describe the one check per pass, the yield and the memory record in place of the in-call wait.
Tests: every InnerRun and notice test listed above that drives a wait inside one Call through `fakeClock.Sleep` drives successive Calls instead, each a fresh producer over the same scratch directory where it models a new pass, and asserts `Yield` where the wait used to keep going; the harnesses (`newRestepHarness`, `newLockHarness`, `newNoticeHarness`, `noticeHarness.producer`) build one producer per pass, and a test that moved the child through `fakeClock.Sleep` calls it itself between two Calls to model the time between passes, so its `onSleep` and `pauseAtSleep` hooks fire between passes rather than inside one; `testInnerRunRefusedResume`, whose `pauseAtSleep` and `onSleep` end its wait inside one Call, drives its checks as successive Calls the same way.
`TestStuckReport_LevelFollowsCause`'s two InnerRun rows, `an InnerRun whose child moves from running to awaiting` (driven by `clock.onSleep`) and `an InnerRun whose blocked child's wait ends on a pause request` (driven by `clock.pauseAtSleep`), drive successive Calls, the first yielding and the next ending with the row's Stuck at the row's level.
`TestNewInnerRun_FloorsPollInterval` keeps only its Warn rows: a sub-second told interval logs one Warn naming `poll_interval_s` and one second logs none, with no assertion on sleeping, since a Call over a running child no longer sleeps.
`TestInnerRun_RunningArmReviewWaitNote` asserts the new running-arm reason texts.
A pass that must respawn models a new process by rewriting the spawn-confirmed marker (`SpawnConfirmedFile`) to a foreign pid before the next Call, since the confirmation is keyed on the pid and one test process would otherwise read its own marker as confirmed.
`TestClearRunMarkers_RemovesEveryRunMarkerSuffixOnly` gains the record's suffix.
battencli's tier1 steps that pin the in-call wait are rewritten to drive passes: `stepStepDrivenRunShed_ReturnsOnTheChildsStateChange`, which asserts a step that blocks and ends Stuck, runs successive passes and expects `Yield` until the child's state change ends a pass Stuck; `stepRunShedPausedChild_WaitsThenTearsDownOnceDone`, which moves the child through `InnerRun.Sleep`, moves it between passes instead; `wireForHub`, which ends the wait through the pause flag after the first sleep, drops that hook, since a pass now ends by itself; and `stepAwaitingChildDoneTeardown_ArchivesTheRunRecords` expects `Yield` from each pass that used to keep waiting, then its `Stuck`.
`TestInnerRun_MemoryOutlivesTheProcess` (tier0, parallel) builds a new producer per Call over one scratch directory and pins only what the record adds: an unposted notice is retried no sooner than one probe apart and dropped after its third attempt across passes, an idle owner's parent lock is released on the pass after the idle bound, a seam failing in every pass warns once and logs Info once when it recovers, a running child with nothing to do yields with batten's status untouched, a done child under a covering pause yields and never returns Done, and a pass that sees the child running followed by a pass that finds it done returns the `child running → done` Stuck before any pass returns Done.

**ImpactSummary:** The Run-Shed producer's whole wait changes shape inside battenshed; its seams gain Sleeps, the running-arm reasons drop their cadence, and every listed InnerRun and notice test, with battencli's four tier1 helpers and steps that pinned the in-call wait, is rewritten to drive passes; the import allowlist gains `internal/state`.
