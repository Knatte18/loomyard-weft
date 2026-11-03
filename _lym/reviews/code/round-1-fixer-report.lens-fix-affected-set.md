```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes for F1–F5 cover every member of the sets their premises name.

Finding, LOW: `setShapedValue` (`internal/yamlengine/set.go:466`), which the F1 change did not reach.
- A shaped entry set whole, such as `tiers.t0={command: {a: b}}` or `build.env={A: {x: y}}`, goes through `openMapEntryNode` with no `mappingShapeError` call.
- A mapping under a text field is written there, and a key the shape lacks is written there too. F1's premise, that the whole-map form is weaker than the per-key form, holds at this second site.
- `mappingShapeError` has one caller, `setWholeOpenMap` (`set.go:383`). This gap predates the fix and the fix did not make it worse.
- The fix covers only the whole open map P, which is what F1 named.

No finding on the other sets:
- **F2:** `setWholeOpenMap` now has one position relative to `shapedOpenMapKey` in both `SetValues` and `SetSparse`.
  - Both loops try `scalarOrListPaths`, then `slices.Contains(openMaps, ...)`, then the shaped-key branch.
  - Those are the only two sites of `slices.Contains(openMaps, pair.Key)` in the set paths. `unknownKeys` at `layers.go:316` is a membership test and `fill.go:108` is unrelated.
- **Parent-split sites:** three sites use `parentMappingOf` (`setWholeOpenMap` and two in `SetSparse`).
  - The sites at `set.go:486` and `set.go:565` still call `splitLastSegment` and `mappingAt` directly. They wrap the `mappingAt` error differently, so the helper is not a fit and they are not misses.
- **F3, F4:** the dotted-map row covers both `SetValues` and `SetSparse`, and the comment test covers `setWholeOpenMap`'s only comment path.
- **F5:** the `labels` once-only assertion sits in the `TestSetSparse` loop, so it runs for every row.

Calls made:
1. `grep -rn "setWholeOpenMap\|mappingShapeError\|parentMappingOf\|splitLastSegment" internal`: to list every caller of the changed functions, which my context did not hold.
2. `grep -rn "scalarOrListPaths\|slices.Contains(openMaps" internal`: to list the other sites of the branch F2 reordered.
