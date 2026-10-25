# Card 13 — innerrun-short-check

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#innerRunProducer`
- `internal/battenshed#NewInnerRun`
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
- `internal/battenshed/doc.go`
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
`Call` keeps its read-before-spawn and spawn path, then calls `once`; `childWait.begun` is the episode's persisted state when one is known, so a state change that happened between passes still ends the Call with its own state-change Stuck and history entry.
`innerRunMemory`, a tolerant JSON record at `innerRunMemoryFile` under the row's scratch directory written through `internal/state`, holds the inventory D4 names: the notice episode (state, since, `arbiterOffAt`, each delivery's attempts, last try, strand-warned and dropped), `driverBusyNoticed`, the owner trigger's `ownedBy`, `ownerNoticed`, `ownerDue` and `ownerCovered`, the `failures` episode ticks (`EpisodeSet.Ticks`/`Restore`), `restepWatch`, and from `childWait` the parent lock's idle readings and owner (`IdleWatch.Waiting`/`RestoreIdleWatch`), `resumeRetryAt`, `notParkedLogged`, `busyLogged`, `busyReason`, `busyKey`, `lastProbe`, `reason`, `reasonKeys`, `pauseWarned`, `coverageWarned` and the revive's start-fallback mark.
`loadMemory` runs at the start of `Call` and a missing or undecodable record reads as empty with one Warn; `saveMemory` runs on every return of `Call` and a failed write warns and changes no verdict.
`runMarkerSuffixes` gains the record's suffix, so a fresh seed clears it with the other run markers.
The owner trigger's own sleep watch (`ownerSleep`) and the idle checks' sleep watch go: `InnerRunDeps` gains `Sleeps func() ([]pauserecord.Interval, error)`, the sleeps the scheduling dobby recorded, which `judgeAgents` and the idle clocks read as excluded time beside the paused intervals (a nil `Sleeps`, or an error warned once per episode, excludes none), and `Awake` is the clock the caller keeps across passes; a wake reaches the owner verdict through the full pass dobby owes, so `foldOwnerTrigger` keeps only its pause-lift and owned-elsewhere-retry triggers.
`reviveDeadDriver` revives once per halt episode, read from the revived marker's presence rather than its pid, since each pass is a new process; the spawn confirmation stays per pass on purpose, so each pass re-checks the child's strand and locks and respawns a driverless running child.
Bound: a file that ends a wait between passes, such as a stop report or a strand's end, is seen at the next pass, up to one `notice_probe_s` later than today, and none is lost.
A yield is logged at Debug (`PATTERN-shed-driver`'s constraint for this task).
Under a covering pause the arm step still does not run, so a done child yields rather than returning Done, and no run enters Land under a pause (D12).
`doc.go`'s InnerRun section, the recipe file's header comment and `docs/overview.md`'s batten sentences on the Run-Shed wait ("waits on its child inside its own call", "waited on without a time limit") describe the one check per pass, the yield and the memory record in place of the in-call wait.
Tests: every InnerRun and notice test listed above that drives a wait inside one Call through `fakeClock.Sleep` drives successive Calls instead, each a fresh producer over the same scratch directory where it models a new pass, and asserts `Yield` where the wait used to keep going; the harnesses (`newRestepHarness`, `newLockHarness`, `newNoticeHarness`) build one producer per pass, and `fakeClock.Sleep` stays only for the in-pass waits that remain.
A pass that must respawn models a new process by rewriting the spawn-confirmed marker (`SpawnConfirmedFile`) to a foreign pid before the next Call, since the confirmation is keyed on the pid and one test process would otherwise read its own marker as confirmed.
`TestClearRunMarkers_RemovesEveryRunMarkerSuffixOnly` gains the record's suffix.
`TestInnerRun_MemoryOutlivesTheProcess` (tier0, parallel) builds a new producer per Call over one scratch directory and pins only what the record adds: an unposted notice is retried no sooner than one probe apart and dropped after its third attempt across passes, an idle owner's parent lock is released on the pass after the idle bound, a seam failing in every pass warns once and logs Info once when it recovers, a running child with nothing to do yields with batten's status untouched, and a done child under a covering pause yields and never returns Done.

**ImpactSummary:** The Run-Shed producer's whole wait changes shape inside battenshed; its seams gain Sleeps, and every listed InnerRun and notice test is rewritten to drive passes, with no caller outside the package beyond the wiring card.
