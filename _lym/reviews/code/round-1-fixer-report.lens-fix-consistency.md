```yaml
lens: fix-consistency
outcome: reported
```

**Task:** fix-consistency lens over the F1–F5 change set (sibling docs and twins of the changed text).

**F-A (LOW)** — `internal/yamlengine/layers.go:157`, the `SetSparse` doc.
It says a shaped open map "takes a key at any path the shape reaches, as SetValues takes it, and a path it does not reach is an error naming the key."
The fix added a refusal of a mapping or list under a text field, and wrote it into the `SetValues` doc (`set.go:118`) and `mappingShapeError` (`set.go:399`).
Both write paths share `setWholeOpenMap`, so `SetSparse` refuses it too; the test row "a whole shaped open map refuses a mapping under a text field" proves it.
Its doc was left unchanged.

**F-B (LOW)** — `internal/yamlengine/doc.go:53` and `internal/configengine/doc.go:100`.
`yamlengine/doc.go:53` says a shaped P refuses "a key inside the mapping its shape does not reach."
`configengine/doc.go:100` says "a path the shape does not reach is refused naming the key."
Neither names the new refusal of a mapping or list where the shape takes a string.
Both were left as they were while `set.go:118` was widened.

**F-C (LOW)** — `setOpenMapEntry` (`set.go:344-368`) and `setPerNameEntry` (`set.go:615-633`).
The fix moves a value's line comment onto the key in `setWholeOpenMap` because a block mapping drops it on marshal.
Both siblings do `mapping.Style = 0` on a template `{}` that may carry the same line comment, and neither moves it.
I didn't run them, so they may or may not lose it.
This predates the fix, so it only counts as a left-behind twin.

**F-D (LOW)** — `setWholeOpenMap`'s doc comment (`set.go:370`).
It says "keeping the old value's comments." That now needs a qualifier.
The new key-move runs only when the new value is a non-empty mapping.
An empty `{}` keeps the comment through `liftEmptyValueComments`.
The reader isn't told that the comment moves to the key, and the second call on an already-written file loses the template's comment.
The fixer worked around that last point by pulling the test out of `TestSetValues`.
The comment documents that limit on the test only, so a reader of `setWholeOpenMap` still can't see it.

No BLOCKING or MEDIUM findings. The branch reorder in `SetSparse` is consistent with `SetValues`: scalar-or-list, then whole, then shaped. `parentMappingOf` is used at all three sites. `TestSetSparse`'s `spec.vars` template and open-map additions don't change any other row's expectations.

Tool calls (1 of 6):
- `grep -rn "does not reach|not reach" internal/*.go` — to find other docs that restate the shape-refusal rule, which my context lacked outside `set.go` and `layers.go`.
