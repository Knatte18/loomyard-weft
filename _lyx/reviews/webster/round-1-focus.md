---
round: 1
focus:
  - "Review range: confirm the live generation's rework round and `head_sha`, then read the branch's own commits after it; check the `cmd/lyx` row in `cmd/lyx/testdata/test-budget.yaml` changed from 74 to 75 and no other row or comment line changed."
  - "Scope: settle whether the range holds anything beyond that one value, since card 12 is the only card and changes nothing else."
  - "Budget guard: settle whether `TestTestBudget_PackagesStayWithinBudget` and `countTestsByPackage` in `cmd/lyx/testbudget_test.go` accept the row both at the branch's own top-level test count and at the merged count."
---

Card 12 is a single-integer edit, so the review is short: range, scope and the budget guard.
