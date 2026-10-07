---
round: 2
focus:
  - "Test budget: `cmd/lyx/testdata/test-budget.yaml` pins the most top-level `func Test...` functions per package, and `cmd/lyx/testbudget_test.go` (untagged, part of `go test ./...`) fails above a row. For every card in `_lyx/plan/` that adds a top-level test function, does the plan raise the matching row in the same card, and do the card's `**Edit:**` targets and `**Intent:**` say so?"
  - "Vocabulary containment: the decision record limits the `5m|1h` TTL vocabulary in `shuttleengine` files to the template defaults, checked under `PATTERN-shuttle-provider-seam`. Does any `shuttleengine` file the plan edits (template comments, `doc.go`, config structs) name the accepted values beyond those defaults?"
---

Both entries are re-checks of the plan as the fixer leaves it; the second also covers any new text the fix adds.
