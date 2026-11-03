## Changes

Commits `b4f7aca1` and the one after it. The five fixes touch the same files, so they share commits instead of one each.

- **F1** fixed as proposed. `mappingShapeError` (`internal/yamlengine/set.go`) returns an error naming the path when the shape takes a string and the node is not a scalar. Refusal rows added to `TestSetValues_Refusals` and `TestSetSparse`.
- **F2** fixed as proposed. `SetSparse` (`internal/yamlengine/layers.go`) now tries the scalar-or-list branch, then the whole-open-map branch, then the shaped branch, as `SetValues` does. The scalar-or-list branch moved up with it, since the whole-map branch would refuse its scalar values. A new helper `parentMappingOf` replaces three inline parent lookups.
- **F3** fixed as proposed. A dotted open map row in `TestSetValues` and a `spec.vars` row in `TestSetSparse` (`sparseTemplate` and its `openMaps` gained `spec.vars`). Only the sparse row creates the parent; the `SetValues` row walks a parent the template holds.
- **F4** fixed with a code change besides the test. The new `TestSetValues_WholeOpenMapKeepsLineComment` failed: a line comment on `labels: {}` was dropped when the replacement was a non-empty block mapping. `setWholeOpenMap` now moves it onto the key. The test is its own function because a file already holding the open map does not carry the template's comment, so the table's second call cannot reproduce the first.
- **F5** fixed with my own change: the proposed `labels` assertion passed without `slices.Compact`, since an empty mapping is not a leaf path in this tree. The assertion counts `rounds`, a scalar leaf that is also a declared open map, and it fails without `Compact`.
- Docs: `SetSparse`, `setWholeOpenMap` and `internal/yamlengine/doc.go` name the text-field refusal and the comment move.

Gate: `lym gate test ./internal/yamlengine ./internal/driftengine` passes.

## Self-check

- fix-code-correctness, MEDIUM: the `labels` assertion pins nothing. Fixed with my own change (count `rounds`).
- fix-consistency F-A, LOW: `SetSparse` doc lacks the text-field refusal. Fixed.
- fix-consistency F-B, LOW: `yamlengine/doc.go` lacks it. Fixed. `configengine/doc.go:100` describes per-key sets, which the refusal does not touch; left as is.
- fix-consistency F-C, LOW: `setOpenMapEntry` and `setPerNameEntry` may drop the same line comment. Deferred: predates the fix and no finding asked for it.
- fix-consistency F-D, LOW: `setWholeOpenMap` doc lacks the comment move. Fixed.
- fix-proposal 1, LOW: the `SetValues` dotted row does not exercise parent creation. Fixed by renaming the row to say it walks an existing parent; the sparse row covers creation.
- fix-proposal 2, LOW: same as F-A. Fixed.
- fix-affected-set, LOW: `setShapedValue` writes a whole shaped entry with no shape check. Deferred: predates the fix, the review named the whole open map only.

## Disputed

None.

## Deferred

- `setOpenMapEntry` and `setPerNameEntry` line comments over `{}` (fix-consistency F-C): outside the review's findings; needs its own check of whether they lose the comment.
- `setShapedValue` shape check for a whole shaped entry (fix-affected-set): outside the review's findings; widening it changes the per-key form's behavior and wants an operator call.
