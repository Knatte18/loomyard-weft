## Changes

Every finding was fixed as proposed unless marked.
The fixes went in one commit, `4a1a7f3c`, instead of one per finding: F1 and F6 both change `internal/gatecli/testmap.go`, and F2 to F10 all change `internal/gatecli/testmap_integration_test.go`.
The self-check's comment fix is a second commit.
A gate round then found four comment lines that wrapped at a fixed column or ran to 120 characters.
Those are rewritten as one sentence per line in a third commit: the `queued` field comment and the `close` doc in `testmap.go`, and the `TestGateTestMapPassYields` and `TestGateTestMapPassSlots` docs in the test file.
No code changed in that round.

- F1: `mapPackages` adds `len(jobs)-i` to `r.queued` on every error return of the job loop and `len(jobs)` on a `Limits` error.
  A plan-loop error or cancellation adds one.
  The signal row of `TestGateTestMapCLI` pins it with the exit record's `Queued`.
- F2: `TestGateTestMapPassSlots` row `APassOnAHubOfOneSlotPlansTakesNoSlotAndCountsEveryJobQueued`.
- F3: `TestGateTestMapPassYields` is now two scenarios.
  `RetakeReplansAndLeavesTheMapAHandCallWrote` asserts the hand call's map is untouched after the retake.
  `RetakeSkipsAPackageAnotherCallLocksAndCountsItQueued` asserts the skip cause and the queued count.
- F4: `TestGateTestMapPassSlots` row `AHeadMoveFoundWhenTheSlotWaitWakesEndsThePassWithTheJobQueued`.
- F5: `TestGateTestMapUpkeep` row `ANamedCallWaitsForABuildLockAnotherCallHoldsAndThenRebuildsThePackageWhole`.
  The wait is injected through `waitBase`.
- F6: `close` writes the exit record only once `r.head` is set.
  `Refusals` now asserts that no refusal leaves a record, and the `writesRecord` field is gone.
  The `PlannedHead` comment and the `gatecli` doc text follow.
- F7: `TestGateTestMapPassSlots` row `ACallInsideAnEnclosingGateRunsSlotTakesNoSlotAndNeverYields`.
- F8: `TestGateTestMapCLI` row `ATerminatingSignalKeepsTheRoundsThatFinishedAndExitsWithItsNumber`.
  Fixed with an own change: the review proposed cancelling the call, which cannot give an exit code of 128 plus the signal number.
  The row runs the command in a child process, through the new `helperRoleTestMapCLI` role, and sends it a real `SIGTERM`.
  A signal to the test process itself would reach every parallel call in it.
- F9: `TestGateTestMapCLI` row `AnUpkeepPassHasNoCapUnlessMaxIsGiven`, through the same child role and the real cobra command.
- F10: two `Refusals` rows, `UpkeepRegistrationReadFails` and `UpkeepRegistrationWriteFails`.
  The second is skipped for root and on Windows.
  Each asserts the refusal's start and its way forward, and the final checks assert that no map or exit record was written.
- F11: the `TestMapExitFileName` comment names the `--upkeep` pass.

Verification: `lym gate test --tier 0 --tier 1 ./internal/gatecli ./internal/testcover ./internal/lymdirs` passed with 0 failures, and the new tests ran.

## Self-check

- fix-consistency, LOW: the `queued` field comment and the `ExitRecord.Queued` comment omit jobs left by an error or signal.
  Fixed as proposed; both comments now name them.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

No fork returned without a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
