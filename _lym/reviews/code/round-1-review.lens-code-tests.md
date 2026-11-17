```yaml
lens: code-tests
outcome: reported
```

**Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.**

Pinned (test → behaviour):
- `TestRun` "ThreeCardPlanRunsTwoBatchesInOrderAndIsDone", "BeginRecordsAreFixedAndLeaseIsReleased", "StaleRecordsAreRemovedAndDoneWritesThemAgain" → the batch loop, begin records (D3) and run entry (D2).
- `TestRun` "FailedAttemptClimbs_*" (unclean tree and removed later target included), "ExhaustedLadderStops…", "MachineryLossRetriesOnce…", "StartFailureIsALoss", "NeedsParentStopsAtOnce…" → D5/D6 attempt failure, retry and climb.
- `TestRun` "FailedStopStartsNothingAndNamesTheGUID", "FailedBatchCommitStopsTheStrand…", "FixerWhoseStopFailedStartsNoNextFixer" → D6 failed-stop and failed-commit paths.
- `TestRun` "PlanConflictAtBeginWritesTheRecord…" → D6 reroute.
- `TestRun` verify and fixer rows (one fixer per failure, flaky rerun, timeout and dirty tree, runner crash, fixer ladder, lost fixer, record commit failure, held lock, wiring error) → D10.
- `TestRun` "FirstBatchBeginRemovesAStaleResetPendingMarker" → reset marker.
- `TestRun_Reentry` → D7: no-spawn done, dirty, failing-gate and row-gates rows each getting a last-rung attempt, `NEEDS_PARENT` rung, card-set mismatch, the lost-last-rung counter, verify re-entry, final commit failure.
- `TestRun_Resume` → D12: recorded-outcome stop, attach by report path, probe error, live fixer attach with delivery, fixer with no record, crash rows.
- `TestReset` → D2: refusals for held lock, absent state, no begun batch, merge, foreign path, live strand, probe error, unstoppable strand, corrupt state with no version, corrupt state with live strand. Plus move/restore/archive/commit, `--stop-strand`, commit failure, marker finish, corrupt-state reset.
- `TestResetVerb` → the CLI envelope and the `--stop-strand` flag.
- `TestActive` → D13: both begun, corrupt Webster, the restore-versus-reset advice, setting errors.
- `TestImplementerGuard` and `TestWeaverProducer_Call` → D1 guard and producer.
- `TestWeaverEntry`, `TestImplementerEntry_*`, `TestLoadConfig`, `TestResolveLadder` → D1/D5 row build and ladder refusals.
- `TestRecordDelivery`, `TestAppendJournal`, `TestRunInFlight`, `TestArchiveRunRecord`, `TestWeaverGeometry` → their named behaviours.

Findings:

1. **MEDIUM, design (test gap).** `internal/weaverengine/run_test.go`: no test runs a rename through Weaver.
   - Behaviour: D3/D11 say a batch's reported rename rewrites the pending cards, and the next partition is built from the rewritten plan.
   - Evidence: grep for `Renames` or `renames` in `weaverengine/*_test.go` and `runplan/*_test.go` returns nothing. The only post-batch-check row is "RemovedLaterTarget" (`run_test.go:240`).
   - The acceptance criterion "after a batch that reports a rename, the next batch is partitioned from the rewritten pending cards; done cards are matched by slug" has no failing row if `report.Renames` (`record.go:173`) or the `Reparse` and re-partition in `beginBatch` (`run.go:322-330`) broke.

2. **MEDIUM, design (test gap).** `internal/weaverengine/run_test.go`: nothing asserts that only the run's true last batch renders as last under a multi-batch run.
   - Evidence: `IsLastBatch` appears in `weaverengine` tests only in `render_test.go:78`, which sets `LastBatch` directly.
   - The "ThreeCardPlan…TwoBatches" row would not fail if `runBatch` (`run.go:405`) or `batchGateSpec` (`strand.go:118`) saw a mapped partition without the pending cards. The acceptance criterion names this check.

3. **MEDIUM, design (test gap).** D6: no test makes the state write fail after an attempt's outcome is recorded.
   - Expected: the strand is stopped anyway and the row returns Stuck naming the fault (`settleAttempt`, `run.go:556-558`; `run.go:491-493`).
   - Evidence: grep for a state-save failure in `weaverengine/*_test.go` finds only a stuck-reason assertion at `run_test.go:320`. The acceptance criterion says "also when the state write fails".
   - Only the failed-stop and failed-commit rows exist.

4. **LOW, design (test gap).** `internal/weaverengine/reentry_test.go`: no row pins `reenter` over the stopped batch's own card edit.
   - Expected: `record.CardHashes` is refreshed (`resume.go:386`) and the edit carries into the next attempt.
   - Only the pending-card edit is named, in "PassingStoppedBatch…AlsoAfterAPendingCardIsEdited" (`reentry_test.go:41`).
   - A row asserting the last-rung attempt's prompt carries the edited stopped-batch card text was not visible in my reads.

5. **LOW, design (test gap).** I found no weaver-specific test of `weavercli`'s `resetDepsFor` wiring.
   - Evidence: `internal/weavercli/*_test.go` holds `reset_test.go` and `testmain_test.go` only.
   - `TestResetVerb` covers the envelope and the flag, not the dep wiring: `StateHistory`, `ReadPlanAt`, `ResetCode` and `DirtyTracked`.
   - The redundancy report lists `weavercli.TestResetVerb` as not judged.

Test-economy notes:
- I did not re-check the report's "contained" candidates (`TestCancellation_RealProducersReturnErrorNotStuck`, `TestGateRows_StopsNameAWayForward`, `shedrecipe/entries_implementer_test.go` rows, `TestWebsterProducer_ConflictRerouteCapThenRepeatHalts`).
- One thing for the handler: `TestImplementerEntry_ForwardsTheRestOfTheRowConfig` has "WeaverGatesKeyBuilds" and "WeaverUnknownKeyIsRefusedByTheConstructor" rows that `TestWeaverEntry` "UnknownKeyIsRefused" and "GatesEntryReachesTheRunDeps" appear to cover (`entries_implementer_test.go:52-53`, `entries_weaver_test.go:69,98`).
- I did not compare their assertions, so I file no finding.

Tool calls (3):
1. `grep` over the weaverengine and weavercli test files, to list test and subtest names; my context held no test files.
2. `grep` for rename, last-batch, state-write-failure, dirty and tier names across the weaverengine, shedrecipe and loomshed tests, to confirm presence or absence.
3. `grep` for renames and save failures, plus `sed -n 100,156p internal/weaverengine/active_test.go`, to read the `TestActive` rows and the `weavercli` test inventory.
