```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the code changed for F1 and F3 (lens fix-code-correctness).

**LOW 1: the reconcile mails the dropped default as kept**
- **Changed text:** the new `{ID: runprofile.ArrasDefaultReinterpretationID, ...}` registration in `internal/configreg/configreg.go`, with `internal/runprofile/reinterpret.go`.
- **Problem:** on the upgrade path in F1, one walk sends two mails that disagree about `defaults.arras`.
- **Evidence:**
  - Take a hub with `defaults.arras: light` and a base holding `solid`. `FollowTemplate` keeps the unit and lists it under `Differs`.
  - `followedLines` (`internal/hubreconcile/walk.go`) turns that into "profiles.yaml kept the hub's defaults.arras: file light, template absent".
  - The reinterpretation then drops the same key in the same write.
  - `walk.go:294-295` mails "defaults.arras: light is dropped" and then "kept the hub's defaults.arras ... keeping every value the operator set ... each kept value is the operator's to settle". The second mail is false for this key.

**LOW 2: `ReinterpretArrasDefault` judges decoded values but removes by literal node**
- **Changed text:** `internal/runprofile/reinterpret.go`, the `removeMappingKey(mappingValue(document.Content[0], DefaultsEntry), shedrun.RecipeArras)` call and the comment about aliases and merge keys.
- **Problem:** the decode follows aliases and merge keys, but the removal does not.
- **Evidence, traced by hand:**
  - If `defaults` itself arrives through a root-level `<<: *x`, the decoded check finds `defaults.arras`. `mappingValue` then returns nil, and `removeMappingKey(nil, ...)` dereferences `mapping.Content` and panics inside the hub walk.
  - If `defaults: *x`, or `arras` arrives through `defaults: {<<: *x}`, nothing is removed. The rewrite still reports "is dropped" and the marker is written, so the file stays unloadable for good.
- **Reach:** this needs hand-written anchors in `profiles.yaml`, so it is unlikely.

**Traced with no defect found**
- **Inputs:** an absent or comments-only file, unparseable YAML (the walk fails with a `FileError`), no `defaults.arras`, a profile that keeps an edited or partial arras block, and an unknown profile name.
- **Added test:** with the registration removed, row 1 of `TestReconcileHubWideAt_ProfilesStillLoadAfterTheArrasBlocksDrop` fails on both the `rewrote` lines and `Load`. Row 2 guards against dropping too much.
- **F3:** removing the duplicate assertion leaves `TestTemplateLadder` correct.

**Tool calls**
1. `grep` of `internal/configengine/set.go` for `JSONEntries`, to check that the dotted `--set` form in the way-forward line exists. My context did not show `Set`. The form is documented there, so there is no finding.
