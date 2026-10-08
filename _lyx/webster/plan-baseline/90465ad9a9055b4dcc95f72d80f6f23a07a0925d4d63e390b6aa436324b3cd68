# Card 4 — rename-multi-name-spec

**Edit:**
- `internal/planglyph#renameDeclSource`
- `internal/planglyph/glyphchain_test.go`
- `internal/planglyph/glyphchain_integration_test.go`

**Intent:**
Fix the second defect the D1 probe found: a `Rename` of one name in a multi-name `var` or `const` spec can never canonicalize.
quarry answers `shapes#PairB` with the whole spec as its signature (`var PairA, PairB = 4, 5`), so `renameDeclSource` derives `var PairA, PairRenamed = 4, 5`, which `quarry.Name` rejects as declaring two symbols, and the plan gate reports a blocking `handle-name-failed`.

For an Old symbol of kind `var` or `const`, `renameDeclSource` derives a one-name declaration from the kind and the new identifier alone, `var <identifier> int` or `const <identifier> = 0`, in the Old symbol's unit.
The derived declaration only feeds `quarry.Name`'s glyph prediction and is never written into the plan, so the dropped type and value cost nothing; a single-name `var` or `const` rename gets the same glyph it got before.

Add the multi-name row's Rename leg (`shapes#PairB` -> `plan:dirname#PairRenamed`) to both glyph-chain files: the plan gate validates clean with the handle canonicalized to `plan:shapes#PairRenamed`, and the delta leg binds it and passes its done-check.
The existing `var`, `const`, block-member and implicit-const Rename legs keep passing on the new derivation.

**ImpactSummary:** Changes the declaration `renameDeclSource` derives for every var and const Rename; the predicted glyph is unchanged for single-name specs and now exists for multi-name ones.
