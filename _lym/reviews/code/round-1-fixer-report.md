# Fixer report, code round 1

## Changes

Each fix is its own commit; F4, F6, F7, F13 and F14 share one, since they all add rows to the testmap integration test.
Gate: tier 0 and tier 1 of `internal/gatecli`, `internal/testcover` and `internal/dobbyengine` pass.

- F4: `TestGateTestMapUpkeep` (`testmap_integration_test.go`) seeds a ghost entry at the old commit into a map and commits a non-body change.
  It asserts the miss, and that the rebuilt map holds no entry measured before HEAD.
- F5: `redundancy_integration_test.go` gives beta's map a `Failure` beside its entry, still judged by the entry.
  drops gains `TestDropsPlain` with a spawn-marked entry over the block `TestDropsLate` covers, which stays unique.
- F6: the stand-in gains `Fails` and `Skips` suffixes.
  A subtest asserts the failed, skipped and no-coverage entries with no coverage.
  Another cancels a call mid-run and asserts no map and no failure is written.
- F7: a subtest gives a kept entry a linked function the fresh build lacks.
  It asserts the entry is re-measured and counted in `self_check_mismatches`.
- F8: `PlanUpkeep` drops the entry of a test not in `Declared` and sets `Changed`; `TestPlanUpkeep` gains `ARemovedTestDropsItsEntry`.
- F9: `PlanUpkeep` clears a `Failure` at another commit and sets `Changed`; `TestPlanUpkeep` gains `AFailureAtAnEarlierCommitIsCleared`.
- F10: `mapPackage` re-runs one kept passing entry (`runFirstKeptEntry`) when no run wrote a profile, so the self-check always has a fresh block table.
  `r.measures` is appended after the dirty and changed-under-tests skips.
- F11: the audit orders an entry with no `MeasuredAt` as the oldest; the `BuiltAt` fallback and its comment are gone.
- F12: the three dobby texts say the refresh plans every map at the prime's HEAD and measures at most `test_map_batch` queued packages.
- F13: the audit subtest gains the no-free-slot guard, the work-queued guard and the equal-re-measure case, each with a positive control.
  The verb's `plan` and `packages` keys are decoded and asserted.
- F14 (partial): the no-free-slot row also shows the plan step holds no slot, since the call returns while the test holds the only one.

- Gate finding: the comment above drops' map in `redundancy_integration_test.go` wrapped at a fixed column.
  It is rewritten one sentence per line.

## Fixed differently

- F10: the `fix` key offered two options, a block table independent of a profile or a counted skipped check.
  Neither is possible without a profile, which only a run writes, and a counter would add a report field.
  The probe re-run reaches the same end.

## Self-check

- fix-consistency, LOW: the `testmap` Long help still said a map with no affected test keeps every entry; fixed in `testmap.go`.
- fix-code-correctness, LOW: the no-profile probe had no row; added `APassWhoseQueuedTestsLeaveNoProfileStillChecksTheKeptEntries`.
- fix-code-correctness, LOW: no row asserted a skipped package reports no measure; added an assertion to `ASkipAfterTheTestsCountsAgainstTheCap`.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- F14, the two fall-back-to-whole-rebuild paths in `planPackage`: reaching them needs a selector verdict that is not a miss beside a test or spawn scan that fails.
  The only way to get that in a test is a hand-built map with a valid format marker and environment stamp, and no fixture builds one.
- F14, the non-whole job skipped when its map no longer stands at HEAD: it needs another call to rewrite the map between the plan and the measure, and `mapPackage` has no seam for that.
  Both would need a test seam in production code, which no finding asks for.

## Rulings

- F1: the parent ruled an equal audit result refreshes the entry's measurement time and kept the code.
  `fillMapTests` already replaces the entry with a fresh one carrying equal coverage and a new `MeasuredAt`, so no change was made.
  The ruling is carried out by the code as it stands, and the parent was not told.
- F2: the parent ruled the dobby's read of the exit record belongs to `testmap-upkeep-dobby` and kept the code. No change was made.
- F3: the parent ruled retiring `test_map_batch` and `test_map_every` belongs to `testmap-upkeep-dobby` and kept the code. No change was made.
  F12's rewording leaves both keys live.
