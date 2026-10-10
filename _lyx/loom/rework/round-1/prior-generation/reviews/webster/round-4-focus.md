---
round: 4
focus:
  - "In cmd/lyx/testdata/test-budget.yaml, check that the cmd/lyx row is at least the number of top-level tests in package cmd/lyx after the merge-in commit d51290bb5, which includes TestRootHook_NoVerboseFlagKeepsTheLevelSetAtInit in cmd/lyx/main_test.go, and that TestTestBudget_PackagesStayWithinBudget passes on the merged tree."
---

The Publish verify failure was the `cmd/lyx` budget row; the focus asks whether the row now matches the merged tree.
