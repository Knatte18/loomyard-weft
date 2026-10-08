---
round: 1
focus:
  - "Fidelity, docs scope: the decision record's scope lists `internal/planparser`'s doc where it lists checks among the same-commit docs; settle whether any card (card 3 names only `validate.go`'s file comment) carries that doc, if the doc lists checks."
  - "Test economy, cards 1, 3 and 5: each plans a new top-level test function (`TestNestedModules`, `TestValidate_VerifyNestedModule`, `TestGlyphChain_CallerCoverageTyped`); settle whether an existing test in the same package already covers the behavior or could take the rows, naming it."
  - "Card order and dependencies across cards 4 and 5: card 5 re-signs `scanReferences`, `coverageSubject` and `coverageSubjects`, which card 4 edits first; settle that card 5's `Edit` glyphs and intent are consistent with card 4's result, and that no card uses something a later card creates."
  - "Verify coverage: the `## verify:` section against every package the cards target, including `contracts/stencils`, `internal/loomcli`, `internal/planparser`, `internal/planglyph`, `internal/websterengine` and `cmd/lyx`; settle that each package's tests run, and that the order matches the plan template."
  - "Attack-surface question, cards 2, 3 and 5: card 3 adds a blocking check and card 5 adds a spawn that degrades to a weaker check; settle that each card's `Intent` states what can now be let through and what bounds it."
  - "Fidelity, scope: each Decision D1–D11 and each Constraint is carried by a card, and no card adds scope the decision record does not license (for example the grep-before-landing step in card 5, and the budget-row edits in cards 1, 3 and 5 against D11's scope)."
---
Focus entries point at where the plan most plausibly diverges from the rubric; none pre-states a verdict.
