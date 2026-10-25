# Fixer report — plan, round 2

## Changes

- **F1 (MEDIUM, design): fixed as proposed, with additions found by the sweep the fix asks for.**
  - (a) Card 7 lists `internal/battencli#TestCancelIntegration_BlockedRunWithNoPair`, `#TestCancelIntegration_KeptPairJudgeOnly`, `#TestCancelIntegration_RemoveTearsDownPairAndBranch` and `internal/pairteardown_test#TestRun_TeardownScenario`; its Tests paragraph says those three tests and the `BoardClaimIsClearedWhenBattenNeverSeededTheRun` row assert `draft`; the ImpactSummary names them.
  - (b) Card 13 lists `internal/battenshed#TestStuckReport_LevelFollowsCause` and `#TestNewInnerRun_FloorsPollInterval`; its Tests paragraph says the two InnerRun rows of the first (`onSleep`, `pauseAtSleep`) drive successive Calls, the first yielding, and the floor test keeps only its Warn rows.
  - (c) Card 4's allowlist sentence adds `gopkg.in/yaml.v3` for `LoadConfig`'s decode, as `internal/battenshed`'s allowlist holds it.
  - (d) Card 27 drops "the one import this card brings in" and says `SpawnDetached` detaches through `proc.Detach`, as `fabricengine`'s detached push does, so the allowlist also gains `internal/proc`.
  - (e) Card 36's allowlist sentence adds `internal/parentdirective`, `internal/editdirective` and `internal/stencil`.
  - (f) Card 17 lists `//contracts/recipes/batten-recipe.yaml` and says its header's Land paragraph drops "names the tasks the landing made ready" and states that a records stop is the one notice of a successful landing.
  - Method sweep: I grepped the repository for every distinctively named symbol on the Delete lists of cards 13 to 39 and checked each hit outside the card's targets against that card or an earlier one.
    It found no new miss: the remaining hits are same-named methods of other types (`sheddriver`'s tackler, `shedadapters`' `handOn`), call sites inside members the card already lists, and `docs/benchmarks/test-redundancy.md`, a measurement report.
    I did not sweep the re-signed and behaviour-changed symbols of every card; I covered the F2 deletion below by hand.
- **F2 (LOW, consistency): fixed differently.**
  - Card 13 lists `innerRunProducer.runningStep`, and its two reason texts drop the cadence (`dobby re-checks it on each probe or child status write`).
  - The deletion of `InnerRunDeps.Sleep`, its `waitOrCancel` default and the `pollInterval` field went into **card 14**, not card 13 as proposed.
    `holdSpawnWhileCovered` (`internal/battenshed/innerrun_pause.go:58`, `p.deps.Sleep(ctx, p.pollInterval)`) reads both until card 14 rewrites it, so deleting them in card 13 would leave battenshed uncompilable at card 13's gate.
  - Card 14 now lists `InnerRunDeps`, `innerRunProducer`, `NewInnerRun`, `pollFloor` and `waitOrCancel`, whose doc names an InnerRun seam.
    It also lists every Sleep fill: `newInnerRunDeps`, `noticeHarness.producer`, `TestNotice_NilNotifyRunsNoStep`, `TestInnerRun_CancelledDuringASeamErrorReportsTheCancellation`, `TestInnerRun_DoneWaitIsNeitherCountedNorBlockedByTheRunningBudget` and `TestNewInnerRun_FloorsPollInterval`.
    The rest of its new targets are `envkit#nilLegal`, `shedrecipe#innerRunEntry` and battencli's `wireForHub`, `stepStepDrivenRunShed_ReturnsOnTheChildsStateChange`, `stepRunShedPausedChild_WaitsThenTearsDownOnceDone` and `shortPollBattenRecipe`, whose doc justifies itself by the seconds a step sleeps and by not faking `Env.InnerRun.Sleep`.
    The search for every `Sleep` fill and every mention of `InnerRun.Sleep` found the sites beyond the review's list.
  - `poll_interval_s` stays as a validated key whose one effect is the floor Warn, as proposed.
  - The proposal had `TestInnerRunEntry_NilSleepIsAccepted` drop its Sleep assertion. Card 14 **deletes** that test instead: its only assertion is the Sleep check (`internal/shedrecipe/entries_batten_test.go:206-218`), and `TestInnerRunEntry_NilNowIsAccepted` already pins that a default env builds.
- **F3 (LOW, consistency): fixed as proposed.** Card 33 creates `plan:internal/dobbyengine#PassExit` -> `type PassExit struct`, and its `SpawnPass` sentence names it as the struct carrying the exit code and last error text; card 37 lists it under Uses.

`lyx loom validate-plan` reports `ok: true` after every change, with only informational findings that predate this round.

## Self-check

- fix-consistency, BLOCKING (card 13 deletes `Sleep` and `pollInterval` while card 14's `holdSpawnWhileCovered` still reads them): fixed as proposed, merged with the identical fix-proposal finding; the deletion and its forced edits moved to card 14.
- fix-proposal, BLOCKING (the same finding): merged into the item above.
- fix-proposal, LOW (the `BoardClaimIsClearedWhenBattenNeverSeededTheRun` row's name and error text still say cleared): fixed as proposed; card 7 renames the row `BoardClaimIsLeftDraftWhenBattenNeverSeededTheRun` and its error text wants `draft`.
- fix-affected-set, LOW (`TestSettleBoardClaim_RunningBattenRecordWayForward` rejects only `write=clear` and `write=done`, so a wrong clear that now logs `write=draft` passes): fixed as proposed; card 7 lists the test and has it reject `write=draft`.
- fix-affected-set, LOW (helper `testInnerRunRefusedResume` ends its wait through `pauseAtSleep`/`onSleep` and is not listed): fixed as proposed; card 13 lists it and its Tests paragraph says it drives its checks as successive Calls.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.
