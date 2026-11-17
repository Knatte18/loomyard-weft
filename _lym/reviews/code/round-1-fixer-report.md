# Fixer report, code review round 1

All twelve findings are fixed.
Tier 0 passes for `./internal/weaverengine` and `./internal/weavercli`, and the new tier-1 wiring test passes.

## Commits

The edits were made in one round, so the commits group findings by file instead of one commit per finding:

- `b223800c` F10
- `9e089ff2` F12
- `dd63657d` F2, F7
- `e7395ee7` F11
- `1e6efc7b` F1, F3, F4, F5, F6, F8, F9
- `ca948e6a` the self-check findings below

## Gate repair

The guard gate failed on `internal/weavercli/wiring_integration_test.go`: a bare `warp` token outside the owner set (`TestEnforcement_FabricVocabulary/tree-scan`).
The test's `warp` variable is renamed `taskTree`.
The guard tier and the tier-1 wiring test pass again.

## Changes

- **F1**, as proposed: `runBatch` takes the stop kind and rung from `nextAttemptRung` when the strand stop fails.
  A needs-parent attempt records a needs-parent stop.
  Row `FailedStopOfANeedsParentAttemptStillRecordsANeedsParentStop` re-enters on the asking rung and spawns.
- **F2**, as proposed: `Reset` settles strands before `refuseForeignPaths`.
  After a stop, every `DirtyTracked` path joins `own`.
  `settleRecordedStrands` now returns whether it stopped one.
  Rows: a refusal that names `--stop-strand` and not `git checkout`, and a reset with `--stop-strand` that discards `live.go`.
- **F3**: row `ReportedRenameRewritesAPendingCardAndTheNextBatchIsBegunFromIt`.
  The fake index's `DetectDrift` now rewrites reported renames into unbuilt cards (`rewriteRenames`).
  The row asserts the card text and the batch-2 card hash.
- **F4**: row `OnlyTheRunsLastBatchIsBuiltAsLastAndTheFirstBatchRefusesARemovedLaterTarget`.
  The fake's `RemovedTargets` gains `removedOnlyWithLaterCards`.
  The row asserts the first batch refuses and the last batch does not, and which prompt carries the main-packages gate call.
  The recorded gate spec has no observable last flag in the fake runner, so the row asserts the prompt alone.
- **F5**: row `FailedStateWriteAfterAnOutcomeStopsTheStrandAndNamesTheFault`, run for a failed and a done attempt.
  A new `onGate` hook on `attemptScript` replaces `state.json` with a directory while the run waits.
  See the Self-check section for the done-attempt change.
- **F6**: `truncateCause` cuts the leading text and the way forward separately, on rune boundaries.
  `FailedStopStartsNothingAndNamesTheGUID` now uses a 1300-byte stop error and asserts the truncation and the `lym reed remove` tail.
- **F7**, as proposed: `Reset` holds the run lock for its whole call.
  Row `HoldsTheRunLockForItsWholeCall`.
  `RunActive` lost its only production caller and was deleted (self-check), and its two test uses were replaced by direct lock probes.
- **F8**, as proposed: `nextAttemptRung` retries a fault attempt on its own rung.
  Row `Crash_FaultAttemptRetriesOnItsOwnRung`.
  The prior-attempt digest of the next prompt now includes a fault attempt, so the retry is told why.
- **F9**: the `beginBatch` doc and the `State.Stop` comment now say a begin leaves the stop record alone.
- **F10**: `weaverengine` is in the bound-packages list.
- **F11**: row `EditToTheStoppedBatchsOwnCardRefreshesItsRecordedHash`.
  The rendered prompt carries card pointers and not card text, so the row asserts the recorded hash alone.
- **F12**: `wiring_integration_test.go` (tier 1) builds `resetDepsFor` over a hub fixture and runs `Reset` through every seam.
  `testmain_test.go` now arms `gitkit.HermeticGitEnv`.

## Disputed

Nothing.

## Deferred

- **F2, the tradeoff behind the proposal.**
  After `--stop-strand` stopped a strand, a tracked edit the parent made by hand counts as the run's own and is discarded.
  The stopped strand's edits cannot be told from a parent's edits made in the same tree.
  I fixed it as the review proposed and documented it in the verb's help and in `weaverengine/doc.go`.
  Whether `--stop-strand` should instead keep refusing foreign paths is an operator decision.

## Rulings

None reached me.

## Self-check

All four forks returned a report.

- fix-proposal, MEDIUM: F2's `own` widening discards a parent's concurrent tracked edit.
  Deferred, above; the consistency finding below is its documentation half.
- fix-consistency, LOW: the `lym weaver reset` help and `weaverengine/doc.go` omitted the F2 behaviour.
  Fixed as proposed.
- fix-consistency, LOW: `RunActive` had no production caller after F7.
  Fixed as proposed: deleted, with its test uses moved to direct `lock.TryAcquireWriteLock` probes.
- fix-proposal and fix-code-correctness, LOW: a done attempt whose end fails left its strand running (the F5 premise).
  Fixed with `settleAttempt` stopping the strand when the end failed, and the F5 row now covers the done case.
- fix-code-correctness, LOW: a failed strand stop of a needs-parent attempt dropped the parent's need from the stuck reason.
  Fixed: the need now leads the reason, and the F1 row asserts it.
- fix-proposal and fix-affected-set, LOW: a persistent fault retries on the same rung on each re-step with no allowance, and the retry was told nothing.
  The no-allowance part is what F8 asked for and each call still stops, so I left it.
  The retry now gets the fault in its prompt digest.
- fix-proposal, LOW: `truncateCause` left a long way forward unbounded.
  Fixed with an own change: the way forward is clipped to the same bound.
- fix-code-correctness, LOW: the F4 row asserts prompt text only and repeats part of `ThreeCardPlanRunsTwoBatchesInOrderAndIsDone`.
  Left as is: the gate spec has no observable last flag in the fake, and the row's new part is the later-reference refusal across batches.
- fix-code-correctness, LOW: the F2 tests pin only the all-own case.
  Covered by the deferred item.
