# Round 3 fixer report — discussion

All three findings validated against the code and fixed.

## Fixed

- **F1** (`quarry.Name` seam): verified that `chokepointGuardedFuncs` in `internal/planglyph/chokepoint_enforcement_test.go` pins `Name` to `CanonicalizeHandles`, and that `handle.go`'s file comment makes `resolvePass` its only caller.
  D5 now routes the head-to-member match through `CanonicalizeHandles`' one batched `quarry.Name` call, the heads a second declaration source under their glyph's `Glyph.UnitPath`, each answer compared with its glyph and kept out of canonicalization, the canonical-collision check and the `RewriteRefs` rewrite.
  Constraints gain the chokepoint pin.
  Support log: Q17 points at the refinement, Q20 records it, and Rejected alternatives gains the second-call-site option.
- **F2** (Scope's rubric bullet): the Scope bullet now names both Plan-Review rubric findings, matching D6.
- **F3** (Rename to-side handle criterion): an acceptance criterion now covers a package self glyph beside a `Rename` whose to-side handle spells a different unit than its Old glyph's, judged by the Old glyph's unit at the plan gate and from `ValidateDispatch`.

## Disputed

None.

## Deferred

None.
