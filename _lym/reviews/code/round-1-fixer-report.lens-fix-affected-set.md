```yaml
lens: fix-affected-set
outcome: reported
```

Task restated: check that the F6 fix and the F1–F5 test fixes cover every site in their sets.

Findings:

1. **LOW**: `internal/configsync/configsync.go:203`, `internal/configcli/doc.go:86` and `internal/hubreconcile/walk.go:218`, with `internal/hubreconcile/doc.go:33` repeating the walk's wording.
   - These four still call `Lacking` "template units the file lacks".
   - After the F6 change, the profiles `Lacking` hook also reports a differing former block as `light.loom: differs from light.tapestry`. That is not a template unit the file lacks.
   - The Warn text "lacks units its template holds, and reconcile never adds them" is therefore wrong for that entry.
   - Its way forward, `lym config <module>`, still covers it, since the operator settles the pair in the editor.
   - The `runprofile` doc comments were updated, but these four sibling descriptions were not.

2. **LOW**: `internal/configcli/configcli.go:629` carries the comment "only a seed-only module whose file lacks a template unit its module reports sets it". This is the same stale wording, now also covering the reported differing pair.

The other sites under the F6 key agree with the fix.
- `internal/yamlengine/rename.go:20` and `:47`, `internal/shuttleengine/config.go:205` and `internal/landingshed/config.go:234` are other modules' own rename hooks, so they are out of scope.
- `recipeRenames` is the only site where `MigrateConfig` and `LackingTemplateBlocks` read the rename table, and both use it.

Sets I did not check: F5's "other walk Warn paths" and F2's "other modules with lacking units". The review names only one module for each, and `configreg.go:145` is the only module with a `Lacking` hook.

Calls made:
1. One `grep` across non-test `.go` and `.md` files for the drop-rule and `lacking` wording. My context lacked the sibling sites outside `runprofile`, `configsync` and `hubreconcile`.
