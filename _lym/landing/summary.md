# gatecli: fix load flake in TestGateTestMap's per-test time limit

`TestGateTestMap/NamedPackageIsRebuiltWhenFresh` could fail a verify gate under load with an unexpected `passed_alone` entry for the healthy fixture test `alpha.TestAlphaTwo`, on branches that did not touch `internal/gatecli`.

The test ran every fixture test under a 3 second limit, the same limit its hanging fixture test had to exceed.
`alpha`'s hanging test also timed out in each call that mapped `alpha`.
A healthy stand-in test running beside it could take more than 3 seconds to start under load,
time out, and then pass when rerun alone.

The change is confined to `internal/gatecli/testmap_integration_test.go`; no production code changes.

Closes #260

## Design decisions

- `alpha`'s hanging test is now `TestAlphaHangsOnCue`.
  It hangs only when `testMapHangingEnv` names a file, and otherwise passes.
  Only the `--all` call that asserts the timeout sets the cue.
- The other calls that map `alpha` therefore no longer report a timeout.
  `NamedPackageIsRebuiltWhenFresh`, and the call that maps `alpha` and `steady` under a forged prebuilt `lym`, now expect a clean result instead of exit 1.
- The per-test limit of the map calls is widened from 3 seconds to 10 seconds, through the new constant `testMapSecondUnit` (500 ms, replacing 150 ms).
  This loosens a check: a healthy stand-in test now has more time before it counts as timed out.
  A call in which a test does hang waits correspondingly longer.

## To check

- The flake was not reproduced, on the base or under load.
  The cause is read from the code and the reported failure, and a pass on a quiet machine does not show that the flake is gone.
- No test fails on the base and passes with this change: the defect is load-dependent timing in test code.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
