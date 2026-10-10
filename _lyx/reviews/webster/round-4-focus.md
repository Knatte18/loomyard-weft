---
round: 4
focus:
  - "In cmd/lyx/testdata/test-budget.yaml, check that every package row the branch raised (internal/loomcli, internal/loomengine, internal/loomrecipe, internal/shedrecipe) is at or above that package's count of top-level tests, and that TestTestBudget_PackagesStayWithinBudget passes when run as lyx gate test ./cmd/lyx."
---

The Publish verify failure is a test-budget row that two independent raises of the same line left one short after the merge-in.
