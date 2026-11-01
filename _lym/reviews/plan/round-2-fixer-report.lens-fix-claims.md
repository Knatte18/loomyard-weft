```yaml
lens: fix-claims
outcome: reported
```

no findings

All the claims the fix added or rewrote hold against the code at HEAD:

- **Load failure without open maps.** Without open maps, a mapping group, a shorter list, or a list whose entries lack a template key fails `configengine.Load` with `missing keys: buttons.task[N]...`. `internal/configengine/config.go:249` calls `yamlengine.MissingKeys`. At `internal/yamlengine/reconcile.go:352`, `sequenceBasePath` exempts a list element only when the path ends in `]`, so a key inside a mapping element such as `buttons.task[0].label` stays required. This matches the reviewer's probe.
- **Any shape loads under open maps.** `MissingKeys` skips any path at or under a declared open map when the layer holds that key (`openMapOf`, reconcile.go). `FillMissing` leaves an open-map key alone whatever its shape (`internal/yamlengine/fill.go:81,108`).
- **A layer replaces the group whole, and an omitted group keeps the template default.** `unitRule.whole` treats an open-map path as one unit in `Merge` (`internal/yamlengine/layers.go:26-27,41`). `FillMissing` adds back a missing open map whole (fill_test row "open map missing is appended whole").
- **`TestModules_SeedOnlyAndHubWideFlags` requires the registry row's open maps to equal the Spec's.** It compares them with `slices.Equal(spec.OpenMaps, m.OpenMaps)` (`internal/configreg/configreg_test.go:66`).
- **"as darn does for `tree_scope`".** darn has `ConfigOpenMaps` at `internal/darnengine/config.go:53`, its Spec sets `OpenMaps: ConfigOpenMaps()`, and its `Modules` row sets `OpenMaps: darnengine.ConfigOpenMaps()` (`internal/configreg/configreg.go:130`).
- **The `TestModules_EveryTemplateKeyClassified` expectations stay valid under open maps.** `LocalDenied` comes from the Categories keys (`configengine.LocalDenied` → `categoryPaths`), so `"buttons"` stays the reed entry. `coveredByEntry` and `holdsPath` match by prefix, so the units `buttons.task`, `buttons.beam` and `buttons.prime` are covered by the `"buttons"` entry.
- **`TestLookup` is tier0.** `configreg_test.go` carries `//go:build tier0`.

Tool calls:
1. Read the `holdsPath`, `coveredByEntry` and `LocalDenied` sources in configreg, to check the classification test against open-map units. My context did not hold them.
2. Read `configengine.LocalDenied` and grepped yamlengine for open-map handling. My context had only the call sites.
3. Grepped `internal/yamlengine/layers.go` for open maps, to check the replace-whole claim in `Merge`. Not in context.
4. Read the build tag of `configreg_test.go`, to check the `tier0` claim for `TestLookup`. Not in context.
5. Read `MissingKeys` and its open-map skip. My context had only its call site.
6. Read `sequenceBasePath`, to check whether list elements that are mappings are exempt. Not in context.
