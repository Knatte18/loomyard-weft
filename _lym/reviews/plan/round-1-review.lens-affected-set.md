```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. Task: for each change a card makes, find what reads or asserts it, and check which cards cover that set.

**Per change: the set found, and the cards that cover it**

1. **The cancel marker is retired (cards 9 and 10).**
   - Card 9 covers: `cancelDeps`, `endRunPhase`, `refuseNothingLeft`, `cancelHold`, `hubCancelDeps`, `startGuard`/`hubStartGuard`/`refuse`, `refuseCancelMarker`, `armSeed`, `preDriveCheck`, `refuseStartOwnedElsewhere`, `DobbyRuns`, pairteardown's `Teardown`/`New`/`registeredBattenAlive`, `TestCancelRun`, `TestStartGuard_Refuse`, `TestPreDriveCheck`, `TestRefuseStartOwnedElsewhere`, `TestRegisteredBattenAlive_ReadsADobbyDrivenRun` and `TestArmSeed_*`.
   - Card 10 covers: the dobby readers `planPass`, `strandedEnded`, `readWaits` and `runNotices`, their tests, `dobbyengine/doc.go`, `shedrun` and `TestCancelIntegration_BlockedRunWithNoPair`.
   - No card covers: `TestArmAt_StartGuardRefusesBeforeAnyWrite`, `cancelFake`/`cancelFake.deps`, `newPreDriveReceiver`, pairteardown's `fakeSubstrates.teardown`, the battencli package doc in `cli.go`, and `pairteardown/doc.go:154` (listed only as Extends).
2. **The cancel keeps the batten record (card 9).**
   - Card 9 covers: `clearRunPhase`, `cancelReport`, `cancelEnvelope` and `cancelNotice`.
   - No card covers: the cancel-order paragraph in `cli.go:19-29`, and `judgeStatus`'s way-forward text at `cleanup.go:542`.
3. **New shed state `cancelled` (card 2).**
   - Card 2 covers: `State.valid`, `stepLocked`, `Goto` and `Relaunch`.
   - No card covers: battencli's exhaustive state switches at `arm.go:530` and `arm.go:902`, and the driver-end switch at `drive.go:164`.
   - Not affected: `battenshed/classify.go:111` covers child states only, and `followup.go:35` is fine.
4. **The PR record path moves to `RunDir` (card 12).**
   - Card 12 covers: `pullRequestDepsGuardedBy`, `prContinueVerb`, `pawlRetiredMove` and `TestPathConstructors`.
   - These take the path from `shedrun` and need no change: `closePairPullRequest` and `TestPullRequestDeps_RecordChangesWatchesTheRunsPullRequestRecord`.
   - No card covers: `TestPawlRetiredMove_CarriesTheScratchPullRequestIntoTheLandingRecord`, whose name and assertions are about the record in scratch.
5. **Fixer attempts move to `AttemptsDir` (card 13).**
   - Card 13 covers: the production members, the fixture constructors `newTestDeps`, `newFinalizeGateFixture`, `newFixerRunFixture` and `newMergeInFixture`, and `TestAcceptFixReview`.
   - No card covers the fixture methods and tests that build attempt paths from the scratch directory: `mergeInFixture.seedAttempts`/`attemptsOnDisk`, `finalizeGateFixture.seed`/`attempts`, `TestMergeIn_Fixer`, `TestFixerRun_Outcomes`, `TestFinalize_GateAndFixer` and `TestFixerAttempts`.

**Findings**

1. **MEDIUM · consistency · card 9 Edit list.** Retiring the cancel marker leaves callers and fixtures outside cards 9 and 10, so card 10's deletion of `cancelmarker.go` breaks the test build. Unlisted members:
   - `internal/battencli#TestArmAt_StartGuardRefusesBeforeAnyWrite` calls `takeCancelMarker` (`startguard_integration_test.go:103-118`).
   - `internal/battencli#cancelFake` and `#cancelFake.deps` set `ReadMarker` (`cancel_test.go:47,141`).
   - `internal/battencli#newPreDriveReceiver` calls `readCancelMarker` (`predrive_test.go:29`).
   - `internal/pairteardown#fakeSubstrates.teardown` sets the seam card 9 renames, `cancelMarked` (`teardown_test.go:117`).

   Fix: add these to card 9's Edit list.
2. **MEDIUM · consistency · card 9 doc targets.** The docs that describe the cancel's order and the marker are not Edit targets of any card.
   - The battencli package doc at `internal/battencli/cli.go:23-29` says the cancel "takes the cancel marker", runs "remove the records", and "remove the marker last". Card 9 makes all three false, and the stray mention at `cli.go:282` also stays. Card 10's grep rule ("leave none in … package docs") cannot be met because no card lists `cli.go`.
   - `pairteardown/doc.go:154` says "no cancel marker of the slug stands". Card 9 must rewrite that sentence but lists the file only under Extends.

   Fix: put `internal/battencli/cli.go` and `internal/pairteardown/doc.go` under card 9's Edit.
3. **MEDIUM · consistency · card 2.** The new `cancelled` state reaches battencli's exhaustive state switches, which no card covers.
   - `arm.go:530` and `arm.go:902` take any state outside {done, running, blocked, awaiting, failed, paused} to "unrecognized status state %q", with no way forward.
   - After card 9, a kept cancel's record is `cancelled`. A `lym batten step` or `goto`, which card 7's act check does not cover, then returns that bare error.
   - `drive.go:164` gives "the driver ended with the run's status at cancelled" when a cancel finishes during a pass.
   - Card 2's ImpactSummary ("refused by step, goto and relaunch only") misses all three.

   Fix: add the `arm.go` switches and `drive.go`'s end classification to card 2 or card 9, with a `cancelled` branch naming `lym fabric remove &lt;slug&gt;`.
4. **MEDIUM · consistency · card 13.** The fixture methods and tests build attempt paths from the scratch directory, and card 13 lists none of them. Card 13 moves the attempt records to `AttemptsDir`, which sits "beside" the scratch directory, and lists only the fixture constructors. Unlisted members:
   - `mergeInFixture.seedAttempts` and `attemptsOnDisk` (`parentalign_fixer_test.go:51,59`).
   - `finalizeGateFixture.seed` and `attempts` (`finalize_gate_test.go:142,150`).
   - `TestMergeIn_Fixer` (`:544,558`), `TestFixerRun_Outcomes` (`fixerrun_test.go:340,426`), `TestFinalize_GateAndFixer` (`finalize_gate_test.go:670`) and `TestFixerAttempts` (`fixerattempts_test.go:21-72`).

   These still pass the scratch directory to `FixerAttemptsPath`. The code compiles, but these tests seed and read the wrong directory. Card 13's claim that they "keep passing through their fixtures" is false.

   Fix: list them under card 13's Edit.
5. **LOW · consistency · card 12.** `internal/loomcli#TestPawlRetiredMove_CarriesTheScratchPullRequestIntoTheLandingRecord` (`pawlwiring_integration_test.go:88`) is not listed. It asserts the PR record carried in scratch, which card 12 moves out of scratch, so its name and assertions go stale.

   Fix: add it to card 12's Edit.
6. **LOW · consistency · card 11.** `judgeStatus`'s skip reason (`cleanup.go:542-553`) says "`lym batten cancel` ends it, and a later cleanup removes it". After cards 9 and 11, a kept cancel's record is refused by cleanup while its pair stands, so the text is wrong for that case.

   Fix: add `internal/battencli#judgeStatus` to card 11's Edit and reword the clause.

**Tool calls**
1. Searched for callers of `shedrun.PullRequestRecord`, `FixerAttemptsPath`/`AcceptFixReview` and the cancel envelope keys; the manifest did not list readers of the PR record or the attempts.
2. Found the enclosing functions of those references; needed to name the missed members.
3. Searched production code that branches on shed states; the manifest did not cover readers of the new `cancelled` state.
4. Read the state branches in `BattenFollowUp`, `skip.go`, the driver rules and `human.go`; needed to judge which of them a cancelled record reaches.
5. Read the exhaustive switches in `classify.go`, `drive.go` and `arm.go`; needed to confirm the default branches.
