---
round: 1
focus:
  - "Card 8 and card 7: does the coverage-guard bridge close? `DiscussionSeats` must leave the allowlist, join `loomrecipe.RecipeEngines` and the engine-set test's want set, and the gate-quiescence guard must treat it as a writer engine with a row over the substituted recipe."
  - "Card 8 and card 3: does `loomrecipe.New` rewrite only the Discussion-Write row's engine, leaving row name, gates, routing and status identity unchanged, and does an absent `discussion_producer` still build the single-agent producer?"
  - "Card 3: does `LoadConfig` normalise null, `[]`, `\"\"` and `[\"\"]` to no advisors, refuse an empty, malformed or unknown entry naming the key and index, load the registry only when the list has entries, and carry the list whole through reconcile?"
  - "Cards 4 and 5: does the chair stencil include the discussion stencil whole without changing its agent-facing body, and does the table builder fill every included block's markers from the one value map without an include that includes?"
  - "Cards 2 and 6: are the new `MultiLLMProducer` and seat-engine seams additive, leaving existing callers' behavior unchanged, and does the fake seat runner evaluate the gate exactly once on a done chair with no `GateOutcome`?"
  - "Per-card mechanical checks: did each card's type-specific checks run and pass against every group's own targets, and does each card stay green on its own as the plan's ordering requires?"
  - "Test economy: do the new tests duplicate an existing test's coverage, repeat a public-surface test per helper, or leave an added behavior uncovered, and does any package pass its row in `cmd/lyx/testdata/test-budget.yaml` without the file edited?"
---

Round 1 seed: the focus follows the plan's cross-card seams, where a defect in one card shows only against another.
Comment-convention compliance is read as part of the base diff review.
