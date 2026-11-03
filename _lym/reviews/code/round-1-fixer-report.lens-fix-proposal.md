```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1 to F5) and look for a concrete case where one is wrong or breaks something.

Findings:

1. **LOW**, `internal/yamlengine/set_test.go`, the new row "whole dotted open map sets its keys under the parent mapping".
   - The F3 proposal asked for a document lacking `a`, to exercise the parent-creation branch.
   - In `SetValues` the working tree is the template, which already holds `a`, so `mappingAt` only walks an existing mapping and never creates one.
   - The row passes with the creating half of the branch broken.
   - Only the `SetSparse` row (`spec.vars`, existing `name: x`) reaches creation.
   - The row's `existing: "name: mine\n"` suggests creation is tested when it is not.
   - Fix: drop the claim, or note that creation is covered by the sparse row alone.

2. **LOW**, `internal/yamlengine/layers.go` `SetSparse` doc comment, near line 157.
   - It describes shaped open maps as taking "a key at any path the shape reaches" and refusing a path the shape does not reach.
   - The F1 change adds a second refusal: a mapping or list under a text field.
   - The fixer updated the `SetValues` doc for this but not the `SetSparse` twin.

The other proposals hold up against the cases I tried:

- **F1:** a null (`command: ~`) and a number or bool under a text field are scalars, so they still pass. A text-shaped open map itself receiving a mapping gets a sensible "takes one string, not a mapping" error. Nothing in the existing rows breaks.
- **F2:** moving `scalarOrListPaths` and the whole-map branch to the top of the `SetSparse` loop matches the `SetValues` order. The scalar-or-list branch stays ahead of the whole-map branch, so `rounds=x` is unaffected. The new `parentMappingOf` helper keeps the same `mappingAt` side effects as before.
- **F5:** the `labels` once-only assertion would fail without `slices.Compact`, because the empty-mapping leaf and the open-map entry sort adjacent. Adding `spec.vars` to `sparseTemplate` and `openMaps` is used only by `TestSetSparse`.
- **F4 (the fixer's extra change, moving a value's line comment onto the key):** the first call keeps `# note`. When the key already has a line comment, it is left alone and the value's comment is dropped, as before. The comment is still lost on a later `--set` of any key, because `carryOpenMaps` overwrites the template value and key comments. That behaviour predates this fix, and the new test records it.

Further tool calls: none. My context already held every changed file.
