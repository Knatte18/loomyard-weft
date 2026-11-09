---
round: 3
focus:
  - "In internal/battencli/lifecycle_integration_test.go, check the startCostBound constant and the step stepStartCost_CommandRowsCostUnderTheBoundBeyondTheCommitAndTheProcess against the cost-budget exception under 'Time is injectable' in pattern/PATTERN-test-speed.md: does the bound sit far above what load adds to the work it measures, and does the constant's comment state the same magnitude as the value?"
  - "Check that the testtiming:keep marker on TestBattenIntegration_Rows still agrees with the startCostBound comment after any change to the bound, and carries no number that can go stale."
---

The round 2 finding on the bound's size is the one open key; the focus asks the fresh reviewer to settle it from the code as it stands.
