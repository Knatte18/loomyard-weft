```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1–F5 fix through the fix-code-correctness lens.

**Finding 1 — MEDIUM**
- Changed text: `internal/yamlengine/layers_test.go`, the `labelsListed` check added to `TestSetSparse` (F5).
- Summary: the check passes with `slices.Compact` removed from `knownKeys`, so it pins nothing.
- Evidence:
  - `collectLeafPathsHelper` (`reconcile.go:480-505`) records leaves only for `ScalarNode`. An empty `labels: {}` is a `MappingNode` with no content, so it adds no leaf and `labels` appears in `Known` once with or without `Compact`.
  - The review's premise that an empty-mapping open map is also a leaf path is false in this tree.
  - A duplicate does occur for a scalar-or-list open map. `rounds: ""` in `sparseTemplate` is a scalar leaf `rounds` and is also in `openMaps`, so `knownKeys` appends `rounds` twice and `Compact` removes one.
  - Counting `rounds` instead of `labels` would fail without `Compact`.

**Correct, run step by step**
- **F1, `mappingShapeError`:**
  - `build={command: {a: b}}`: the shape has `Fields`, `command` is `Text`, and the node is a mapping, so it errors with `build.command takes one string, not a mapping`.
  - Both new refusal rows fail without the Text check, because the old code returned nil.
  - `command: ~` is a scalar and still passes.
  - A `Text` entry in a map of names (`env`) holds scalars and still passes.
  - A non-text single value still returns nil.
- **F2, `SetSparse` reorder:**
  - `scalarOrList` now runs first, then the whole-map branch, then the shaped branch, matching `SetValues`.
  - A scalar-or-list path such as `rounds=x` never reaches `setWholeOpenMap`, so it is not refused as "not a flow mapping".
  - `parentMappingOf` gives the same parent creation and errors as the old inline code.
  - No test pins the order, but the review says no current input reaches the difference.
- **F3, parent-creation rows:**
  - Without the `parentPath` branch, `setEntry(root, "b", …)` would write a top-level `b:`, and both rows' exact or contains assertions would fail.
  - The `SetValues` row's existing file lacks `a`, but the working tree is the template, so `a` is already there and the branch only walks it.
  - The `SetSparse` row creates `spec` from nothing.
- **F4, `setWholeOpenMap` comment move:**
  - A non-empty replacement moves the old value's line comment onto the key, which gives `labels: # note\n    bug: desc\n`. The new test fails without that move, as the first gate run showed.
  - An empty `{}` replacement keeps the comment on the value, and that case marshals correctly.
  - If both the key and the value carry a line comment, the value's is still lost. This is an unlikely input.
  - After one set, the file's key comment is not carried into a later `SetValues`, because `carryOpenMaps` copies only the value and the marshalled key comes from the template. The comment is lost on the second set. That behaviour was there before the fix, and the new test's doc comment states it.

**Calls**
1. Read `reconcile.go:496-536`: `collectLeafPathsHelper` was not in my context, and I needed to know whether an empty mapping is recorded as a leaf.
