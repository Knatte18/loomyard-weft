---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
---

# Fixer report

## Changes

- F1: removed the planning clause from the `internal/testcover/` line of `docs/overview.md`.
- F2: the second `meets` test is now `TestMeetsSecondMeetsTwo`, with blocks `[0, 1]` against the first's `[0]`.
  The stand-in now waits for the other test after writing its profile, not before, so a shared profile path shows in the entries.
  Corrected the comment on the alpha assertion.
- F3: added the `TheTiersParallelIsWhatGetsCapped` row and the `EachTierRunsAsManyTestsAtOnceAsItsOwnParallel` subtest.
- F4: added an empty commit carrying the two `Loosened-Check:` trailers.
- F5: both warnings now open `gate test --map:`.
- F6: the template comment names a coverage build for a map run, coverage and `lym` builds for the redundancy report step,
  and coverage, mutant and `lym` builds for `lym gate redundant`.
- F7: `fillMapTests` appends the run entries to the existing ones; `reindexed`, `ranNames` and the kept-entry indexing are deleted.
- F8: `runTestMap` computes its scope under `preTierCeiling` and refuses through `preTierExpiry`.
  The test requests now set the ceiling.
- F9: the dirty-linked-package subtest asserts that no coverage run happened.
- F10: `AnEllipsisTargetMapsEveryPackageBelowIt` maps `outcomes/...` beside the new `outcomeskin` package.
  The fixture is in that test rather than the linked-packages one.
- F11: added `APackageWithNoMapAndAnUnparsableTestFileIsMappedAsFailed`.
- F12: the stand-in records run order; the tier0 test now sorts after the tier1 one and must run first.
- F13: the task-worktree test covers a failed listing beside a dirty file inside and outside `backend`.

## Verification

`lym gate test --tier 0 --tier 1 ./internal/gatecli ./internal/gateslot` exited 0 for both tiers.
The tier1 log shows the `TestGateTestMap*` tests passing, the new subtests included.
The new and changed tests were not run against a deliberately broken implementation, and F8 has no test of the expiry.

## Disputed

## Deferred

## Rulings
