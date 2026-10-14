# planglyph: delete-order skips references the deleting card rewrites

The delete-before-reference check no longer reports a reference to a deleted member when an Edit target of the deleting card, or of a card numbered before it, covers the line that holds the reference.
That card rewrites the code before the later card runs, so the reference is gone by then.

Before, `deleteOrderFindings` asked only whether a later card's Edit region held the reference.
A reference added after Plan-Write by an earlier batch, such as a temporary parity test the deleting card itself edits away, blocked begin-batch with `plan_drifted` and no override.
This is what blocks `overview-seats`.

## Design

- The exemption lives in `deleteOrderFindings` (`internal/planglyph/deleteorder.go`).
  Both callers share it, the plan-gate `callerCoverageFindings` and the scan-only `LaterDeleteReferences`, so the two paths agree.
  It takes a second region list, `allRegions`, used only for the exemption; findings are still named from the later cards' `regions`.
- A line is exempt only when a region of a card numbered at or below the deleting card holds it.
  A line only a later card's region holds is still reported, and the finding wording is unchanged.
- `LaterDeleteReferences` now also builds regions for the deleting cards, since its webster begin-batch caller passes the batch's own cards as `deleting` and they otherwise get no regions.
  Those regions are dropped from the reporting list afterwards.
  The early return when no later card has an Edit target stays.
- `editRegion.holds` factors out the line-in-region test that `laterEditRegions` also uses.
- The quoted-import arm (`laterImportFindings`, for a package self glyph Delete) keeps no such exemption.
  `doc.go` states this.

## To check

- Building regions for the deleting cards resolves their Edit member targets too.
  An Edit target of a deleting card whose answer is unreadable now raises a `glyph-rejected` finding at begin-batch where it did not before.
- Tests are new rows in the `TestLaterDeleteReferences` table: the deleting card's own region, an earlier card's region, and a reference only a later card covers.
  `lyx gate test ./internal/planglyph` was the targeted run; the full `./...` and `--tags integration` tiers were not run.
- After landing, resume `overview-seats` with `lyx loom resume` in its pair.
