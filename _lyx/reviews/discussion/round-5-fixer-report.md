# Round 5 fixer report — discussion

Every finding's premise was checked against `contracts/specs/loom-plan-spec.md`, `internal/websterengine/doc.go` and `internal/planglyph/deleteorder.go`; all hold, and all seven are fixed.
`lyx loom validate-discussion` exits 0 after the fixes.

## Fixed

- **F1** (Rename vs spec mechanic): D5 drops the `Rename` Old side from caller coverage, because the spec's AST-aware Rename rewrites every reference itself; its bound is that mechanic, its grep verify and the build gate.
  The spec's Rename text is unchanged.
  Scope, acceptance criteria, Q6 and Q21 are aligned, Q23 is added, and the rejected alternative is logged.
- **F2** (clearing remedy vs D3): D5 states that on the deleting or re-signing card, D3 leaves only a member glyph whose span holds the reference, or another file, as covers.
  For a `Delete`, a package self glyph or the declaring file works only from an earlier card.
  References in the declaring file outside the member's own span count as callers.
  The over-report clearing sentence, the caller-coverage criterion and the Open risk now say this, and Q24 is added.
- **F3** (file/package Delete): D5 states that file-path, file-self-glyph and package-self-glyph `Delete` targets are not checked.
  Their bound is the build gate, plus `delete-before-reference`'s import match for a package.
  The file-Rename Old side falls under F1's exclusion.
  An acceptance criterion and Q25 are added.
- **F4** (arrow on interface methods and fields): D5 limits the arrow to members a single Go declaration names.
  An interface method or struct field takes no arrow, with a stated bound.
  D6's missing-arrow rubric finding exempts them, the arrow criterion lists them as blocking targets, and Q26 is added.
- **F5** (own package): D5 defines a member's own package by the declaring directory plus a matching package clause, so an external `_test` package is matched by the qualified form.
  A criterion and Q27 are added.
- **F6** (two-card re-sign): D5 states that each card is checked against the plan-time tree, and the criterion now has each card covering the callers.
- **F7** (changed-files guard): D3's Bound now rests on a redundant listing being plan noise.
  The support log's rejected alternative is restated on the premise that a card's targets are fixed at plan time, before the probe finds the defects.

## Disputed

None.

## Deferred

None.
