# Round 1 fixer-report

## Changes

- F1 (fixed as proposed, commit `acea0f96`): `TestMidMerge_FabricParkedWithConflicts_Parked` in `internal/fabricengine/midmerge_integration_test.go` now builds its fixture with `newMergeLayerFixture(t, &clashLayer)` in place of the three hand-repeated steps.
- F2 (fixed as proposed, commit `71837acb`): the Cleanup `Expect` comment in `internal/fabricengine/livestate_verbs_test.go` now reads "Build moves the prime pair off f.OriginalBranch,".

`go vet ./internal/fabricengine` is clean, and `lym gate test ./internal/fabricengine` passed tier0 (921 passed, 0 failed).

## Self-check

Every fork returned a report, and none returned a finding.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
