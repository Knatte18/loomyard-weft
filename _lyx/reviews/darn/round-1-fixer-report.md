## Changes

All three findings fixed as proposed, in one commit (`0331e1931`) because the edits share files; `lyx gate test ./internal/yamlengine` is green.

- **F1**: replaced `isScalarOrListOpenMap` with `scalarOrListOpenMapPaths` (`internal/yamlengine/set.go`), which reads the template's own value at each open-map path.
  `SetValues` and `SetSparse` call it before `carryOpenMaps`, so only a path the template declares as a scalar or list takes the new branch.
  Added a refusal row to `TestSetValues_Refusals` and one to `TestSetSparse` (template `labels: {}`, file `labels: [a]`, pair `labels=x`).
- **F2**: `scalarOrListNode` refuses a value whose trimmed text does not end at `]`.
  Added the `["a"] trailing` row to `TestSetValues_Refusals` and `TestSetSparse`.
- **F3**: added the empty-list and nested-list rows to `TestSetSparse`, so the change description's claim holds, rather than editing `_lyx/landing/summary.md`, which this round must not touch.

## Disputed

None.

## Deferred

None.
