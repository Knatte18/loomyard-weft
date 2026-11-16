---
round: 3
focus:
  - "internal/gatecli/redundant_integration_test.go, TestGateRedundant: does any subtest wait out a production-second duration, with `redundantCmd` in internal/gatecli/redundant.go setting `secondUnit` to `time.Second`, rather than a short injected unit, per PATTERN-test-speed's Time is injectable rule? Does the fixture show that `mutationCheck.mutantTimeout` applies `mutantTimeoutFloor` below `redundancy_test_timeout_sec`, so the floor and the test bound give different values?"
  - "internal/testcover/sourcescan_test.go, TestScanSourceReaders_MarksTestsReachingTheScanKit: does each walk order of the call cycle behind the scan kit get reached deterministically, with no loop over random map iteration order, so a regression in one order always fails the test?"
---

Both entries name where to look and the question to settle; the reviewer judges the current tree.
