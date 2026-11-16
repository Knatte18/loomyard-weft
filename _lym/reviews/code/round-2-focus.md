---
round: 2
focus:
  - "internal/gatecli/redundant.go runRedundant and internal/gatecli/redundantmutants.go mutationCheck.run: does the Covering list printed for a candidate or a refuted test equal the testcover.CoveringTests pick its mutation verdict was judged against, and does TestGateRedundant assert that list exactly?"
  - "internal/gatecli/redundantmutants.go runMutants cover pass: is a covering test's kill that was made beside other runs, including a cover that is also a candidate, rerun alone before JudgeMutants reads it?"
  - "internal/gatecli/redundant_integration_test.go cov fixture: does a test reach buildMutants' drop-and-rebuild loop, the //lym:guard exclusion in sourceCheckingTests, and the mutant timeout bound with a timed-out run counted as a kill?"
  - "internal/testcover/sourcescan.go ScanSourceReaders: are a call cycle and a selector call on an imported package handled so a test is neither wrongly judged nor wrongly left unjudged, with a row in TestScanSourceReaders_MarksTestsReachingTheScanKit for each?"
---

The focus covers the areas the fixes of this round touch; it caps nothing and states no verdict.
