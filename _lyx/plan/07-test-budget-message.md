# Card 7 — test-budget-message

**Edit:**
- `cmd/lyx#budgetViolations`
- `cmd/lyx#TestBudgetViolations`
- `cmd/lyx#TestTestBudget_PackagesStayWithinBudget`
- `pattern/PATTERN-test-speed.md`

**Intent:** Make the test-budget guard's failure name the row value to set, decision D6, so a post-merge collision of two one-row raises is a one-line edit.
`budgetViolations`'s over-budget message keeps the package, its count and its budget, and ends by telling the reader to set the package's row in `cmd/lyx/testdata/test-budget.yaml` to the count; its no-row message tells the reader to add a row for the package with the count to that file.
`TestBudgetViolations`'s rows assert the new exact strings.
`TestTestBudget_PackagesStayWithinBudget`'s `t.Errorf` suffix no longer repeats the file, and says only that the edit lands in the commit that adds the tests.
`pattern/PATTERN-test-speed.md`'s test-budget bullet says the guard names the value to set.
The guard's logic, the file format and the same-commit rule are unchanged.

**ImpactSummary:** Message text only, in one guard and its table-driven test; no package budget changes.
