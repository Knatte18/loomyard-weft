---
round: 1
focus:
  - "internal/planglyph/deleteorder.go `deleteOrderFindings`: does the skip cover exactly the deleting card and cards numbered before it, with a line held only by a later card still reported? Check the card-number comparison for off-by-one and for the deleting card being one of several."
  - "internal/planglyph `LaterDeleteReferences`: building Edit regions from the deleting cards as well as later ones, each card once. Does a card appearing in both sets get duplicated or mis-numbered, and does the new unreadable-answer path at begin-batch behave as the summary's Risk section claims?"
  - "`insideLaterEditRegion` and `laterEditRegions` after the `editRegion.holds` extraction: is the caller-uncovered hand-off meaning unchanged?"
  - "Test coverage: do the new `TestLaterDeleteReferences` rows and the changed `TestGlyphChain_CallerCoverage` row pin each of the three cases (own Edit, earlier card's Edit, only-later-card still reported), and does the package stay within its `cmd/lyx/testdata/test-budget.yaml` row?"
  - "Docs: the `internal/planglyph` doc entry states the exemption and the `laterImportFindings` non-exemption accurately against the code."
---

Seed focus only; the reviewer judges the whole diff against the board entry.
