# Card 1 — plan-spec-card-model

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Intent:** Reword the card model in `loom-plan-spec.md` so a card carries coverage for its behavior rather than a test file of its own (D2, D3, D4).

- "What a card is", criterion 3: replace "Bundles its own test, when it introduces new behavior (implementation + `_test.go` in the same card)" with a one-clause criterion: a card that introduces behavior carries the coverage for that behavior in the same card, by extending an existing test where one covers the surface and by a new test only for behavior no existing test covers, per `PATTERN-test-economy`.
  Keep "Pure refactors/renames may rely on existing tests instead."
- "Card granularity": "One card per independently reviewable and testable unit" becomes "One card per independently reviewable unit".
  The private-supporting-type sentence drops "or testability": "A symbol with no independent meaning apart from another symbol in the same card …".
  "A symbol that is independently testable or reusable gets its own card …" becomes: a symbol that is part of a module's public surface, or reused across packages, gets its own card even when one card is its first consumer; a helper serving one surface is bundled into that surface's card.
- "Worked example" intro: Card 2 is described as the multi-label example only (an `**Edit:**` group followed by a `**Create:**` group); drop "for the implementation-plus-its-own-new-test-file shape".

The worked example's fenced card bodies stay byte-identical, since `internal/planparser`'s golden fixture mirrors them.
After the edit, none of these phrases remain in the file: "Bundles its own test, when it introduces new behavior", "independently reviewable and testable unit", "independent meaning or testability", "independently testable or reusable", "the implementation-plus-its-own-new-test-file shape".
No test pins this file's wording: the only claims on the old phrases sit in `claims_test.go` rows for the stencils cards 2 and 3 edit, so no test joins this card.
The new lines follow semantic line breaks and point at `PATTERN-test-economy` rather than restating it (`PATTERN-producer-pointer-rule`).

**Commit:** 1: plan-spec: card coverage and granularity follow the public surface
