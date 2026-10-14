# Decision record — delete-order-own-edit

## Goal

The delete-before-reference check stops reporting a reference to a deleted member when the deleting card, or a card numbered before it, edits the code that holds the reference, since that card rewrites the code before the later card runs.

## Scope

- `internal/planglyph/deleteorder.go`: `deleteOrderFindings` and `LaterDeleteReferences`.
- A test in `internal/planglyph/deleteorder_test.go` pinning the exemption and the unchanged report.
- `internal/planglyph/doc.go`'s delete-before-reference entry states the exemption.

Out of scope: `laterImportFindings`, the quoted-import arm for a package self glyph Delete, which the board entry does not name.

## Decisions

- **The exemption lives in `deleteOrderFindings`.**
  A resolved or name-only reference line that lies in an Edit region of a card numbered at or below the deleting card is skipped before the later-region lookup.
  Rationale: both callers, the plan-gate `callerCoverageFindings` and the scan-only `LaterDeleteReferences`, share that function, so the two paths agree.
  Bound: the exemption skips only a line that an Edit region of the deleting card or an earlier card holds; a line only a later card's region holds is still reported, and a skipped line inside the plan gate stays covered for caller-uncovered by the admissible-target rule for a deleted member.
- **`LaterDeleteReferences` builds regions from the deleting and later cards together, each card once.**
  Rationale: the webster begin-batch caller passes the batch's own cards as deleting and every unbegun card as later, so the deleting card's own Edit regions are otherwise never built; an earlier card already done has run its edit and needs no region.
  The early return when no later card has an Edit target stays, since no reference can then be reported.

## Constraints

- PATTERN-test-economy: the new behavior is new rows in the existing `TestLaterDeleteReferences` table, not a new top-level test.
- PATTERN-comment-line-breaks: Go comments one sentence per line.
- No change to the finding wording, which webster and tests parse.

## Auto-mode assumptions

- A darn run has no interview; scope and calls come from the board entry and a reading of `internal/planglyph`.
- "Any card before it" means any card numbered below the deleting card among the cards the caller passes.

## Open risks

- Building regions for the deleting cards resolves their Edit member targets too, so an Edit target of a deleting card whose answer is unreadable now raises a glyph-rejected finding at begin-batch where it did not before.

## Acceptance criteria

- A reference that a later card's Edit region holds, and that the deleting card's own Edit region also covers, is not reported.
- A reference that a later card's Edit region holds, and that an Edit region of a card numbered before the deleting card also covers, is not reported.
- A reference that only a later card's Edit region holds is still reported as blocking delete-before-reference.
- `lyx gate test ./internal/planglyph` passes.
