# Card 12 — cmd-lyx-budget-row-after-merge

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `cmd/lyx/testbudget_test.go`

**Intent:**
PR-Gate finding F1: the `cmd/lyx` row in `cmd/lyx/testdata/test-budget.yaml` is raised from 74 to 75, and nothing else changes.

The branch's review-fix commit for the root hook's log level added `TestRootHook_NoVerboseFlagKeepsTheLevelSetAtInit` to `cmd/lyx/main_test.go` and raised the row from 73 to 74.
`main`, in the webster audit gaps commit, added one `cmd/lyx` top-level test of its own and raised the same row from 73 to 74 with a textually identical edit.
The two edits merge clean, so the merged package holds 75 top-level tests against a row of 74, and `TestTestBudget_PackagesStayWithinBudget` fails on the merged tree.

Set the row to 75: the merged count, which `countTestsByPackage` in `cmd/lyx/testbudget_test.go` reaches once `main` is merged in.
The merge with `main` is Finalize's own catch-up merge-in, not this card's, and the plan session runs no git merge.
`budgetViolations` passes a count below its row, so the raised row holds on this branch, where the package has 74 tests, and on the merged tree, where it has 75.
Edit only that row's value; every other row and the file's comment lines stay as they are.

Test: none new.
`TestTestBudget_PackagesStayWithinBudget` is the test that pins the row, and it runs under the package's own gate.

**ImpactSummary:** One integer in one budget row; no Go source changes, and the only test that reads the file is the budget guard, which passes at and below the row.
