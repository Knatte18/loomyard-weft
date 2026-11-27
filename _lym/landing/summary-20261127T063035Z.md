# TestGateTestMap: hang alpha's fixture only on cue, under a 10 s limit

`TestGateTestMap` (`internal/gatecli`, tier 1) could fail under load with an unexpected `passed_alone` entry, `alpha.TestAlphaTwo`, on a branch that touched nothing it covers.

## Cause

Every map call in the test ran its fixture tests under one per-test limit: `gate.redundancy_test_timeout_sec`'s 20 seconds of 150 ms each, 3 seconds.
alpha's hanging fixture test had to exceed that limit, and it hung in every call that mapped alpha, beside `TestAlphaTwo` in tier1.
A stand-in fixture test is a re-exec of the whole `gatecli` test binary, and under a loaded tier run its start can outlast 3 seconds.
`TestAlphaTwo` then timed out beside the hanging test, was rerun alone, passed, and was listed as passed alone.

## Change

The fix is in `internal/gatecli/testmap_integration_test.go` alone; no production code changes.

- The stand-in `go` gains a `HangsOnCue` suffix: such a test hangs only when `testMapHangingEnv` names a file, and passes otherwise.
  alpha's hanging test is now `TestAlphaHangsOnCue`.
  `AnAllCallMapsEveryPackageWithATestAndRecordsAFailedBuild` sets the cue and still pins the timed-out outcome and map entry;
  every other call maps the test as a pass and waits out no limit.
  `WhileATestHangs` already sets the cue, and the `Hangs` suffix that the redundancy and CLI tests rely on is unchanged.
- The map calls' test second is the named constant `testMapSecondUnit`, 500 ms, which makes the fixtures' 20 seconds a 10 second limit.
- `NamedPackageIsRebuiltWhenFresh` and `AMapRunBuildsNoLym` now expect a clean call, exit 0 with no timed-out or failed test, instead of the incidental timeout.

The hanging test now costs one call two 10 second waits, beside `TestAlphaTwo` and then alone, where every call that mapped alpha waited two 3 second limits before.

## Design decisions

- A per-test limit split in production code was not taken: the defect is in the test's timing, and the cue keeps the fix in test code.
- The red-first gate cannot judge this fix, as the parent recorded under "No test seam: the flake is in the test's own time limit":
  the changed test passes on the merge base on a quiet machine.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
