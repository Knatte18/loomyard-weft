```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each card's change, I listed what reads or asserts it, and which cards cover that set.

**Per change: the set found, and the cards that cover it**

- **Cancel marker replaced by the cancel intent (cards 9 and 11).**
  - Production readers: `armSeed` (arm.go:246), `preDriveCheck` (arm.go:765), `refuseStartOwnedElsewhere` (owner.go:151), `startGuard.refuse` and `hubStartGuard` (startguard.go:285, :87), `cancelDeps.ReadMarker` and `TakeMarker` (cancelcmd.go:251), `DobbyRuns` (dobbyreads.go:187), `Teardown.New` and `cancelMarked` (teardown.go:196, :725), and dobby's `planPass`, `drivePasses`, `strandedEnded`, `readWaits` and `runNotices`.
  - Tests: `TestCancelRun`, `TestStartGuard_Refuse`, `TestArmSeed_CancelMarkerRefusesTheWrite`, `TestRefuseStartOwnedElsewhere`, `newPreDriveReceiver`, `TestPreDriveCheck`, `TestArmAt_StartGuardRefusesBeforeAnyWrite`, `TestCancelIntegration_BlockedRunWithNoPair`, `TestCancelMarker`, `TestJudgeCancelMarker`, `TestPathConstructors`, `TestPlanPass`, `TestStrandedEnded`, `TestRunNotice_Conditions` and `TestWaitReminders`.
  - Covered in full by cards 9 and 11.
- **The cancel keeps its record and reports `records_removed` only under `--remove` (card 10).**
  - Asserted by `TestCancelRun` (cancel_test.go) and by `TestCancelIntegration_KeptPairJudgeOnly`, `RemoveTearsDownPairAndBranch` and `BlockedRunWithNoPair`.
  - Covered by card 10.
- **A record now persists with the shed state `cancelled` (cards 3 and 10).**
  - Readers: `battenPreStepUnmarked`, `clearDoneRun`, `judgeStatus`, `BattenFollowUp`, `slotsTaken` and `checkRunStatus` (arm.go:533).
  - `checkRunStatus` is held: card 12's `refuseByAct` runs before it, and its default branch still refuses an unknown state.
  - `battenshed.isLegalState` (classify.go:111) reads child states only, so it does not apply.
  - Not covered: `DobbyTasks`, which sets a released task's start refusal (dobbyreads.go:361–376). See F2.
- **The PR record moves into `RunDir` and every write publishes (card 7).**
  - Write sites: `recordPullRequest`, `updateRecord`, `closeRecordedPullRequest`, `openPullRequest`, `dropRecordedPullRequest`, `openAtOnce`, the write in `prContinueVerb` and `pawlRetiredMove`'s `RemoveScratch`. All are covered by card 7.
  - Readers that take the path through `shedrun.PullRequestRecord`, so the move carries them:
    - pairteardown/pullrequest.go:59
    - prsession.go
    - landingdeps.go:348 and :432
    - `TestPullRequestDeps_RecordChangesWatchesTheRunsPullRequestRecord` (watchfilechanges_test.go:132)
  - Not covered: `TestPawlRetiredMove_CarriesTheScratchPullRequestIntoTheLandingRecord`. See F3.
- **Fixer attempts move from `deps.ScratchDir` to `RecordsDir` (card 8).**
  - All production sites are covered.
  - Tests that build the path from `ScratchDir` themselves are only partly covered: card 8 lists only the fixture constructors. See F1.
- **`_lym/loom` gains files.** reportengine/stops.go:97 reads only the subdirectories and the friction directory there, so the new files do not affect it.

**Findings**

- **F1 — MEDIUM, consistency.** Card 8 (`08-fixer-attempts-tracked.md`) misses test members that build the attempt path from `deps.ScratchDir`.
  - Card 8 lists only `newFixerRunFixture`, `newMergeInFixture` and `newTestDeps`. These members read or seed `FixerAttemptsPath(...ScratchDir...)` directly:
    - `finalizeGateFixture.seed` (internal/landingshed/finalize_gate_test.go:142)
    - `finalizeGateFixture.attempts` (:150)
    - `TestFinalize_GateAndFixer` (:670)
    - `mergeInFixture.seedAttempts` (internal/landingshed/parentalign_fixer_test.go:51)
    - `mergeInFixture.attemptsOnDisk` (:59)
    - `TestMergeIn_Fixer` (:544, :558)
    - `TestFixerRun_Outcomes` (internal/landingshed/fixerrun_test.go:340, :426)
  - Once production reads `RecordsDir`, these tests seed or read the wrong directory, unless the fixtures alias `RecordsDir` to `ScratchDir`. That aliasing would void the card's own "nothing under `ScratchDir`" assertion.
  - Fix: list these members under `**Edit:**` and switch them to `deps.RecordsDir`.
- **F2 — MEDIUM, scope.** Cards 10 and 11 leave dobby's start path blind to a kept `cancelled` record.
  - `DobbyTasks` (internal/battencli/dobbyreads.go:361–376) sets `TaskView.StartRefusal` only for a `done` record.
  - Suppose a released task still has a kept `cancelled` batten record. Dobby then starts it every tick, and each start is refused by card 12's `refuseByAct` (cancelled). Those refusals pile up into failing-pass notices instead of a start refusal shown in the view (dobbyengine/view.go:72 and :126).
  - No card lists `DobbyTasks`, or its test, as an affected reader of the new persisted state.
- **F3 — LOW, consistency.** Card 7 does not list `TestPawlRetiredMove_CarriesTheScratchPullRequestIntoTheLandingRecord` (internal/loomcli/pawlwiring_integration_test.go:88).
  - The test seeds the record through `shedrun.PullRequestRecord` and drives `pawlRetiredMove`. That function's removal now publishes, which commits and pushes, and the record is no longer scratch.
  - The card changes both the test's premise and its name, yet lists the other loomcli tests it touches.

**Tool calls**

1. grep of the tests that name the cancel marker, `records_removed`, `PullRequestRecord` and `FixerAttemptsPath`: the manifest had the production callers only.
2. Mapping of those test hits to their enclosing functions, to compare against the cards' Edit lists.
3. The `FixerAttemptsPath` and PR record lines in those tests and the gate fixture: needed to tell whether they build the path from `ScratchDir`.
4. grep of the exhaustive `shedengine` state switches and the dobby tests that set the marker: needed the readers of the new `cancelled` state.
5. classify.go `isLegalState` and arm.go `checkRunStatus`: the two switches the previous grep surfaced.
6. grep of `LoomDurableDir` readers and of `StartRefusal`, plus reportengine/stops.go:85–120: needed the readers of `_lym/loom` and the dobby start-refusal path.
