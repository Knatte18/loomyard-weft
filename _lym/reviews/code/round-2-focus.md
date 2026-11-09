---
round: 2
focus:
  - "In pattern/PATTERN-test-speed.md under 'Time is injectable', read the bullet this branch adds on measured-time comparisons and any exception it carries; settle whether internal/battencli/lifecycle_integration_test.go's startCostBound step (stepStartCost_CommandRowsCostUnderTheBoundBeyondTheCommitAndTheProcess) obeys that rule exactly as written."
  - "In internal/battencli/lifecycle_integration_test.go, settle whether the testtiming:keep marker on TestBattenIntegration_Rows and the startCostBound comment state the same bound and purpose."
---

Both entries target the branch-tip commits that reworded the rule and the marker, which no reviewer has yet seen.
