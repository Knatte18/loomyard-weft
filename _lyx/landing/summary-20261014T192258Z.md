# Delete-before-reference skips code the deleting or an earlier card edits

The delete-before-reference check reported a reference to a deleted member whenever a later card's Edit region held it, even when the deleting card, or a card before it, edits that same code first.
That blocked `overview-seats` at Webster: card 18 deletes `internal/loomengine#PlanSpec` and edits `TestPlanTable`, which removes the call, yet card 19's Edit region over the same test made begin-batch refuse with `plan_drifted`.

## What changed

- `deleteOrderFindings` (`internal/planglyph/deleteorder.go`) skips a reference line that an Edit region of the deleting card, or of any card numbered before it, also holds.
  A line only a later card's region holds is still reported as before.
  Both callers share this function, so the plan-gate path (`callerCoverageFindings`) and the scan-only `LaterDeleteReferences` path agree.
- `LaterDeleteReferences` now builds Edit regions from the deleting cards as well as the later ones, each card once.
  Webster's begin-batch passes the batch's own cards as deleting and every unbegun card as later, so the deleting card's own regions were never built before.
- `editRegion.holds` replaces the containment test that `laterEditRegions` spelled inline, since the new lookup needs it too.
- The `internal/planglyph` doc entry for delete-before-reference states the exemption.

## Design decisions

- The exemption sits in `deleteOrderFindings` rather than in `laterEditRegions`, because `insideLaterEditRegion` builds on the latter for the caller-uncovered hand-off and must keep its meaning.
  A line skipped at the plan gate stays covered for caller-uncovered, since a deleted member already admits its own card's and earlier cards' targets.
- `laterImportFindings`, the quoted-import arm for a deleted package, keeps no such exemption; the doc entry says so.
- A done earlier card is not consulted: the caller does not pass it, and the tree the check reads already holds its edit.

## Risk

Resolving the deleting cards' Edit member targets means an unreadable answer for one of them now raises a glyph-rejected finding at begin-batch where it did not before.

## Tests

- `TestLaterDeleteReferences` gains rows: a reference the deleting card's own Edit covers is not reported, one an earlier card's Edit covers is not reported, and one only a later card covers is still reported beside the deleting card's own Edit.
- `TestGlyphChain_CallerCoverage`'s row that pinned the old report for a reference the deleting card's own Edit covers now expects no finding.
