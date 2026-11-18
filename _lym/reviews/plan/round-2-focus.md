---
round: 2
focus:
  - "Cards that add a gate-slot holder (the `lym gate mutate` verb, and any map refresh run inside a plain `lym gate test --gate` call): does each card edit `internal/gateslot/doc.go` and `pattern/PATTERN-gate-slot.md` so the holder list names the new holder, as `PATTERN-gate-slot` requires?"
  - "Batch boundaries: for each production function a card creates in `internal/testreach`, `internal/gateslot` or `internal/testcover`, is a caller reachable from a `main` package in the same card, given tier1 `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` runs at every batch gate and the `batch` key runs tiers 0 and 1?"
  - "Any card whose edit changes a string the template config or a test helper matches by exact text (for example the `code-burler-fix` line in `internal/loomcli/landingdeps_verify_test.go`): is every such matcher listed under **Edit:**?"
  - "Every refusal or budget branch a card prescribes (the `lym gate reach` range cap, the `lym gate mutate` `reach_budget_min` and `mutation_budget_min` branches): does a named input and a named test row reach it, and does the card state the outcome?"
  - "Decision fidelity for triage in the prime: does card 11 follow the decision that a call with no parent branch has no change set, or is a departure recorded in the decision record?"
---

Round 2 re-checks the areas where round 1 found gating or structural defects.
