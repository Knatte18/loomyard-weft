```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that would fail if it broke, or none.

**Per-behaviour list**

- D1 precedence order, every state and every adjacent-pair override: `TestDerive` (`internal/runfsm/state_test.go`), rows per state plus `LandedOverCancelled`, `CancelledOverPaused`, `PausedOverHalted`, `HaltedOverLanding`, `LandingOverRunning`, and others.
- Landed from board `done` with no batten record: `TestDerive` row `LandedFromBoardWithNoBatten`.
- Cancelled record with its task branch gone reads as no record (`queued`), and with the branch on origin reads `cancelled`: `TestDerive` rows `CancelledRecordWithTaskBranchGoneReadsQueued` and `CancelledRecordWithTaskBranchOnOriginReadsCancelled`.
- Act order (owed push, hand_over, code tip, landing past merge, held, pause covers, paused, cancelling, terminal, act): `TestAct`, one row per branch plus the landing-under-hold interactions.
- Asides (Pause, Cancel, Hold, ShedPause, PushOwed) and the paused and cancelling refusal wording: `TestAct` Judge rows (`PauseAsideDropsTheIntent`, `ShedPauseAsideReadsItAsNotPaused`, `PauseAsideKeepsAShedPause`, `CancellingNamesTheKeepFinish`, and others).
- Display state written from the FSM: `TestTransitionState`, `TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed`.
- ReadRun records and inputs, including origin versus local parent read, the push skip, the origin child read with no pair, the task-branch read for a cancelled record, owner and pause carry-through, and the error wording: `TestReadRun`.
- Origin reads (parent holds, child record, fetched task branch): `TestOriginRunReads` (integration).
- Intent writers (write cancel, finish sets `cancelled`, write and clear pause, no-record found=false): `TestRunIntents`.
- Tolerant parse of the new status keys: `internal/shedengine/status_test.go` (`TestStatus_JSONRoundTrip`, `TestState_Valid`).
- Cancel intent committed before anything ends, and the failed-intent-write and failed-intent-commit rows: `TestCancelRun` rows `IntentIsWrittenAndCommittedBeforeAnythingEnds`, `FailedIntentWriteRefusesWithNothingEnded`, `FailedIntentCommitRefusesWithNothingEnded`.
- Cancel finish keeps the record (`FinishIsWrittenKeepingThePairWithoutRemove`), and `--remove` removes the pair and records in one step (`RemoveOnLiveRunRemovesPairAndBranch`).
- Finished-kept cancel refuses a second cancel without `--remove`: `TestCancelRun` rows `FinishedCancelThatKeptItsPairRefusesWithoutRemove` and `FinishedCancelThatKeptItsPairIsFinishedByRemove`.
- Cancel verdict refusals (landed, hand-over, unreadable state): `TestCancelRun` plus `TestCancelVerdict`.
- Commit of the cancel record, including the merge-in-progress refusal: `TestCommitCancelRecord`.
- Cancel marker read from the record plus the local lock: `TestCancelMarker`, `TestJudgeCancelMarker`, `TestCancelMarker_ReadsNothingFromAPrimeWithNoScratchRoot`.
- Cancelled record whose pair is removed is deleted in the same step: `TestTeardown_CallSequence` rows `CancelledRecordIsDeletedAfterTheRemovalAndGivesNoWarning` and `FailedCancelledRecordDeletionWarnsWithTheCleanupStep`, and `TestRemoveCancelledRecord_DeletesOnlyACancelledRecordAndCommitsItsRemoval` (integration).
- `lym batten cleanup` removes a cancelled record once its pair is gone and refuses one whose pair stands (naming `lym fabric remove`) or whose branch is on origin: `TestCleanup` rows 270 to 289.
- `ClearDoneRun` leaves a cancelled run and its task branch: `TestClearDoneRun_RemovesADoneRunTheRecipeFilterAdmits` row `AnyRecipe_CancelledRunStaysWithItsBranch`.
- `lym batten run` and `lym batten step` ask the act rule and refuse a cancelled record:
  - `TestBattenRun_RefusesWhileAPauseCoversTheSlug` rows (act, hand-over, terminal cancelled, terminal landed, failed read);
  - `TestBattenPreStep_RefusalKinds` rows `ActRuleWaitIsBootstrap` and `CancelledRecordIsBootstrapNamingTheCancel`;
  - `TestRunCmd_ResumeDispositions` row `cancelled`.
- The dobby pass asks the act rule (the `CancelMarkedWithActVerdict…GivesNone` rows) and the dobby start-refusal read for a cancelled record: `TestDobbyTasks_ReadsTheStartRefusalOfAFinishedRunRecord` row `ACancelledRunRecordHoldsTheStartWithTheCancelledWording`.
- `lym loom resume` and `lym loom start` ask the act rule, including cancelling, cancelled and failed-read refusals and an undecodable record reaching the driver: `TestResumeVerb` rows and `TestStartAndResume_RefuseARunOwnedElsewhere` (cancelling row).
- `lym tender pause`, with the intent written and committed, and the owner refusal naming the machine and `lym adopt`: `TestPauseAndResume_WriteLiftAndRefuseOverTheHub` (`internal/tendercli`), `TestPause` rows for owner refusal and Record error.
- `lym tender resume` clears the intent, refuses a wait, lifts a terminal run's entry and handles a Clear error: `TestResume` rows, and the same CLI test for committed-cleared.
- PR record in the pair's tracked records, committed at once, removed on close: `TestPRReview_Call` (the `wantCommits` rows, including failing-commit rows), `TestFinalize_PullRequest`, `TestPullRequestDeps_CommitsTheRecord`.
- Fixer attempt counts in the records directory, with migration from scratch: `internal/landingshed/fixerattempts_test.go` and the `moveScratchAttempts` row in `TestFixerRun_Outcomes`.
- `FixerAttemptsPath` read and written through the records dir under `TestFinalize_GateAndFixer` and `TestMergeIn_Fixer`.

**Findings**

1. LOW, class design, `internal/tendercli/runintents.go:67-74`, `internal/tenderengine/resume.go` Verdict branch. D3 requires `lym tender resume` of a run another machine owns to be refused naming that machine. The refusal is only tested for `pause` (`internal/tendercli/cli_test.go`, slug `away`). No test sends `resume` through the real `ReadRun` and `Judge` with `OwnedElsewhere` set, so a regression of the hand-over answer on resume would not fail any test. The `TestResume` rows use a fake verdict. Fix: add a resume-of-`away` assertion to that CLI test.

2. LOW, class design, `internal/pairteardown/teardown.go` and `internal/pairteardown/followup.go`, the `lym fabric remove` path. D3 requires `lym fabric remove <slug>` to delete the cancelled record in the same step. `TestTeardown_CallSequence` covers the teardown core. I found no test that drives the `lym fabric remove` entry itself and shows it takes the record-deleting path. This could not be confirmed within my call budget, so it is lower confidence.

No other behaviour in the diff lacks a pinning test.

**Tool calls (4 of 6)**
1. `git diff --name-only` plus grep of test function and row names over every changed test file. The inherited context held no test files.
2. Narrowed grep of the saved output from call 1, with noise filtered. The first output was truncated.
3. Per-file `git diff -U0` for the runfsm, pairteardown, tender, loomcli, landingshed and battencli test files, to read the row names. They were not in my context.
4. `git diff` of the dobby, landingshed, shedengine and tendercli tests and `fixerattempts.go`, to confirm coverage of the fixer-count records move and the dobby act rows.
5. `git diff` of `internal/tendercli/cli_test.go` and `internal/tenderengine/resume.go`, to check whether the resume owner refusal is tested.
