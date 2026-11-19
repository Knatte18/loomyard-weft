---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
---

## Changes

- F1, commit `7d7a5598`: `diffMutation.run` in `internal/gatecli/mutatecheck.go` returns the outcome with `runs` set, through a new `findingsSoFar`, on the cancelled-context and `runPackage` errors.
  The slot error returns `d.outcome`, which is the empty outcome the call seeded.
  `mutationCall.run` in `internal/gatecli/mutate.go` assigns `found.outcome` before the error check.
  `recordFindings` still records no Functions or FileHashes on error.
- F2, commit `3cc84bba`: `TestMutationStep_RecordFindings` gives `found` a pre-check not-judged entry, and its outcome an unjudged mutant and a not-judged package.
  Every row expects the Unjudged list and the NotJudged list, the pre-check entry first.

- Gate repair: the gate found no mapped test reaching `freshMaps` or `fillMutateAnswer`.
  `internal/gatecli/mutate_test.go` adds `TestFreshMaps_SortsPackagesIntoJudgedAndNotJudged`, with a worktree row and a prime row, each covering fresh, stale, unmapped and unrefreshed packages.
  It also adds `TestFillMutateAnswer_ListsTheCheckAndTheNotJudgedPackagesOfBothStages`.
  The gate's other lines list reaching tests and map ages, and needed no change.

`lym gate test ./internal/gatecli` passed (tier0, 0 failed), after the repair too.

## Self-check

- fix-code-correctness, LOW: no test fails if F1 is reverted, since the recorder test builds its findings by hand. Deferred: the round-2 review's Rejected section declines an integration row, because the round-1 fixer found no deterministic failure point after a package has produced findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- The fix-code-correctness LOW above: a test that fails a later package's run has no deterministic failure point, as the review's Rejected section records.

## Rulings

None.
