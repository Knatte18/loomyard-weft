---
round: 1
focus:
  - "Card 2 cites rows 33 and 35 of `contracts/specs/loom-plan-spec.md` by number, while card 1 inserts a new validation row after `verify-package-missing`: settle whether card 1's insertion shifts those numbers, or whether the rows are better named by check ID."
  - "Card 1 `Edit:` member glyphs: settle with `lyx quarry resolve` that `internal/planparser#ledger`, `internal/loomcli#reworkParityFixture` and `internal/loomcli#loomCLI.validatePlanCmd` resolve, and that every changed signature (`reworkParityFixture` gains `edit`, `coverageVerbAndWay` in card 2 gains `files`) carries the re-sign arrow while members whose bodies alone change carry none."
  - "Card 3 edits `contracts/stencils#wordingClaims`: settle that the glyph resolves to the file that holds it, and that the claims on the plan stencil and the Plan-Review rubric it names exist as the Intent quotes them."
  - "Card order: card 2 leans on card 1's `edit-test-file` wording and card 3 on both; settle whether any card uses something a later card creates or changes, naming the card pair."
  - "Fidelity to the answer key: settle that D1 to D6 and the `Added after Discussion` decision, the Constraints, and the acceptance criteria each map to a card, and that no card carries scope the decision record does not license (card 2's `keepFile` test field and new `RedundantFile` row, card 1's `Glyph.UnitPath` doc rewrites)."
  - "Test economy: settle whether the new `TestGateParity_PlanGate` and `TestGateParity_ReworkPlanGate` rows and the new `TestIndex_AppendsPlanChecks` rows duplicate what `TestCheckEditTestFile` already covers, naming the covering test for any that do."
  - "Reachability of refusals: card 1's parse-error skip and path-shaped-ref skip, and card 2's `_test.go` carve-outs, each need a named input and test that reaches the branch; settle that each named input survives upstream checks such as `glyph-malformed` and `ParsePlan`'s canonicalization."
  - "Verify coverage: settle that `## verify:` runs the tests of every package the cards target (`internal/planparser`, `internal/planglyph`, `internal/loomcli`, `contracts/stencils`, `cmd/lyx`) and follows the template's section order."
---

Seed focus only: each entry names where to look and a question to settle, and none pre-states a verdict.
