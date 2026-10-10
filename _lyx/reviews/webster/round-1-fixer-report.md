# Webster round 1 — fixer report

## Fixed

- **F1** (LOW, consistency) — validated: counting top-level `func Test…` in each package's `_test.go` files, `TestMain` excluded, gives `internal/shedengine` 133, `internal/proc` 17 and `internal/loomcli` 182, below their rows of 135, 18 and 183.
  Set the three rows in `cmd/lyx/testdata/test-budget.yaml` to those counts.
  `lyx gate test ./cmd/lyx`, which runs `TestTestBudget_PackagesStayWithinBudget`, passes.
  Commit `31c1d8b9f`.

## Disputed

None.

## Deferred

None.
