---
round: 1
focus:
  - "Cards 2 and 4: does the diff retire `skip_merge_review` everywhere (code, `loom.yaml` defaults, recipe, docs) and migrate existing values, and does a perch budget of 0 keep the writer → Bouncer route with the Bouncer settling at once?"
  - "Cards 1, 4 and 5: do loom and darn resolve budgets only through `internal/reviewbudget`, with null inheriting and 0 meaning zero rounds for perches while hard caps keep 0 as inherit, and does darn refuse per-run budget keys by layer?"
  - "Cards 7, 8 and 9: is `internal/arbiterstop` the single home of the eligibility rule (only `hard-cap` and `circling` escalations), with no copy in `battenshed`, the Go driver or batten, and does batten's hold release only when the arbiter has handed on?"
  - "Card 10: does the driver stencil stay recipe-blind, send a twice-failed transient to its fork before the arbiter, and relay no notice after a resume?"
  - "Test economy: do new tests per card duplicate an existing test's coverage, and does any package exceed its row in `cmd/lyx/testdata/test-budget.yaml` without that file in the card's `Edit:` list?"
  - "Docs-in-same-commit: do changed modules update their `doc.go`, and does `PATTERN.md` or `docs/overview.md` carry the new cross-cutting invariants where the plan requires?"
---

The focus follows the plan's card order and the two cross-cutting risks: budget semantics (0 versus null versus inherit) and the single-home eligibility rule.
