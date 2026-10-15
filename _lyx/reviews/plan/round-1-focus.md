---
round: 1
focus:
  - "Fidelity: settle whether card 4 carries every part of the live generation's F1 fix (host-chosen judge part, second plain part restored from `main`, `planLensDirective` and `discussionLensDirective` values, the stencil tests, `docs/overview.md`, `internal/twilldirective/doc.go`) and the decision record's last entry, and whether any other scope (for example the `internal/loomengine/doc.go` or `twill.go` comments that still describe the critical fixer at both hosts) is left uncarried."
  - "Card 4 Intent and Targets: settle whether each prescribed test change names a glyph in `Edit` or `Uses` that its Intent actually edits (`TestDirective_*`, `TestStencils_TopLevelMarkers`, `wordingClaims`, `TestDiscussionTable`, `TestPlanTable`), and whether each `Edit` member glyph that changes a signature carries the re-sign arrow."
  - "Reachability: settle whether each refusal row the Intent prescribes (empty `RejectionRule` with no critical judge, empty `GoverningInput` or `DisputeEvidence` with one, a missing judge stencil) has a named input and a named test row that reaches it, and that no upstream check in `fillPart` or `mergedValues` rejects that input first."
  - "Test economy: settle whether the claimed 'no new top-level test' holds, and whether any new row duplicates a case an existing test already pins."
  - "Verify: settle whether `## verify:` runs the tests of `contracts/stencils`, `internal/twilldirective` and `internal/loomengine`, and follows the plan template's section order."
---

Single-card plan; card order and granularity are low risk.
The card's Intent restates the F1 fix at length, so the main question is whether its claims about `main`'s bytes and the existing test rows match the code.
