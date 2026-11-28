```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the F1–F4 change set.

1. **LOW**: the anchor guard added for F2 (`internal/runprofile/reinterpret.go`, `holdsAnchor` check in `ReinterpretJacquardBlocks`) covers jacquard blocks but not the sibling writes to `defaults.jacquard`.
   - In `reinterpretJacquardDefault`, `setMappingValue(defaultsNode, jacquardRecipe, …)` replaces the old scalar node, and `removeMappingKey(defaultsNode, jacquardRecipe)` deletes it. Neither checks `Anchor`.
   - Example: `defaults: {tapestry: light, jacquard: &d solid, darn: *d}`, where `light` and `solid` hold the blocks they need. The set path removes the node carrying `&d`, so `*d` is left dangling and the written file does not parse. This is the failure F2 describes.
   - The new doc line scopes the guard to "a jacquard block", so no stated claim is false. Only a hand-written anchor reaches this case.

2. **LOW**: the F3 half-drop guard (`holdsBesideOwnKey` before the drop in `reinterpretJacquardDefault`) has a twin in the same file that was left as it was.
   - `ReinterpretArrasDefault` still calls `removeMappingKey(mappingValue(document.Content[0], DefaultsEntry), shedrun.RecipeArras)` and reports "is dropped" whenever an own key was removed.
   - With `defaults: {<<: {arras: x}, arras: x}`, where `x` has no arras block, the merged value survives, so the line is untrue and the file still does not load. Its doc says a default written through a merge key "is flagged for a hand edit, and the file is left as it is".
   - This function predates the fix and the fix did not touch it, so the rubric may put it out of scope.

I found no other disagreement between the changed text and its surroundings: the doc comments, the `doc.go` paragraph, the flag wording, the new test rows and the `TestTemplateLadder` doc after the F4 deletion.

Tool calls: none.
