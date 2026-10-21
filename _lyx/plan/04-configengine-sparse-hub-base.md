# Card 4 — configengine-sparse-hub-base

**Create:**
- `plan:internal/configengine#SparseBaseHeader` -> `const SparseBaseHeader = "# This file holds the hub's tuned values over lyx's template.\n"`
- `plan:internal/configengine#baseSparseMarkerName` -> `const baseSparseMarkerName = "base-sparse"`
- `plan:internal/configengine#BaseSparseMarker` -> `func BaseSparseMarker(boardDir string) string`
- `plan:internal/configengine#BaseSparseMarkerRel` -> `func BaseSparseMarkerRel() string`
- `plan:internal/configengine#sparseBase` -> `func sparseBase(dirs Dirs, spec Spec) bool`
- `plan:internal/configengine#BaseScaffold` -> `func BaseScaffold(dirs Dirs, spec Spec) string`

**Edit:**
- `internal/configengine#Spec`
- `internal/configengine#BaseLayer`
- `internal/configengine#load`
- `internal/configengine#Set`
- `internal/configengine#Edit`
- `internal/configengine/doc.go`
- `internal/modelspec#ConfigSpec`
- `internal/runprofile#ConfigSpec`
- `internal/configreg#Module`
- `internal/configreg#TestModules_SeedOnlyAndHubWideFlags`
- `internal/configengine_test#TestLoad_LayerStack`
- `internal/configengine_test#TestSet`
- `internal/configengine_test#TestEdit`
- `internal/configengine_test#TestConfigPaths`
- `pattern/PATTERN-config-strictness.md`

**Uses:**
- `internal/configengine_test#newLayerFixture`
- `internal/configengine_test#layerSpec`
- `internal/configengine_test#newBase`

**Intent:**
`configengine` learns that a hub's base layer is sparse over the template (D2), writes it sparse (D4's `--set` and editor bullets) and stops logging its template fill (D5).

`Spec` gains a `SeedOnly bool` field, documented as the module whose file is the operator's registry, seeded whole and never rewritten, so its base stays full in a hub too.
`modelspec`'s and `runprofile`'s `ConfigSpec` set it, and `TestModules_SeedOnlyAndHubWideFlags`'s check that each module's `Spec()` carries the entry's own flags also compares `spec.SeedOnly` with `m.SeedOnly`, so a registry entry and its spec cannot disagree.
`sparseBase` reports whether the base layer under dirs is sparse: `Board` is told and the spec is not `SeedOnly`.

`SparseBaseHeader` is D2's header comment, the whole content of a base file with no tuned unit; it is one comment line saying the file holds the hub's tuned values over lyx's template.
`BaseSparseMarker` returns `<boardDir>/_lyx/config/base-sparse`, the tracked board marker D3 names, and `BaseSparseMarkerRel` its path relative to the board dir, declared beside `SparseMarker` (`PATTERN-lyxdirs-single-declarer`); it has no `.yaml` suffix, so no module reader takes it for a module file.
`TestConfigPaths` gains rows for both, including the row that `BaseSparseMarker` is `BaseSparseMarkerRel` under the base, and one that it differs from `SparseMarker`.

`load` no longer logs the "filled missing keys from template" Info line when `sparseBase` holds, since a sparse base leaves most keys to the template; a standalone stack and a seed-only module keep the line, and the unknown-keys warning and every refusal stay as they are.
A present base file holding only `SparseBaseHeader` already reaches the stack through `readLayers`, which drops a comment-only file only for the machine and local layers, so it counts as a present base under `Load` and `LoadOrTemplate`; this card adds no code for that but pins it.
`TestLoad_LayerStack` gains a `seedOnly` field on its rows; its hub-geometry rows that today expect a fill line ("template fills every depth the layers lack", "template fill is logged with the layer template") expect none, a copy of one with `seedOnly` set expects the line, a standalone-geometry row expects it, and a new row with a header-only default file loads under both loaders with no refusal, no fill line and every value from the template.

`Set` writes a base layer for which `sparseBase` holds through `setSparse`, so only the keys set are written and an absent file is created holding only them, never scaffolded from the template; a seed-only or standalone base keeps today's template scaffold, and a `JSONEntries` spec keeps `setEntries`.
`TestSet` gains rows: a hub default-layer write into an absent file writes exactly the key set (`wantExact`), a hub default-layer write into a present sparse file keeps its own units beside the key set and adds no template key, and a seed-only hub default write into an absent file still scaffolds the template.
`BaseScaffold` returns the bytes an absent base layer is scaffolded with: `SparseBaseHeader` when `sparseBase` holds, else the spec's template.
`Edit` scaffolds an absent base through `BaseScaffold`, and `lyx config`'s hub-wide editor, the one production editor of a hub base, uses the same function (card 10), so the scaffold rule lives in one place; its empty-save abort is unchanged, so a save keeping the header leaves a key-less base (the header is a comment, not empty).
The file header comments of `set.go` ("The base layer scaffolds from the template like Edit does") and `edit.go` ("Scaffolds missing files from a template") say a hub base is written sparse and scaffolded with the header, and only a seed-only or standalone base from the template.
`TestEdit` gains rows on the default layer under a board: an absent hub base shows the editor `SparseBaseHeader` (`wantSaw`) and a save leaving it is kept as the header-only file, and a seed-only one still shows the template.

`BaseLayer`'s comment and `internal/configengine/doc.go` drop "stays full": in a hub the base layer is sparse over the template, a key-less base holds `SparseBaseHeader` and loads as a present base, its fill is not logged, `Set` and `Edit` write it sparse, and only a seed-only module's base and a standalone worktree base stay full.
`configreg`'s `Module` comments that speak of seeding the base (`SeedOnly`, `SeedsFromPrime`) say the seed of a non-seed-only base is sparse.
`pattern/PATTERN-config-strictness.md`'s fill bullet says a key the stacked layers lack resolves to its template default, and that the fill is logged only for a standalone stack and a seed-only module, since a hub's base is sparse; its strictness bullet adds that a header-only base is a present base.

**ImpactSummary:** the hub base layer's write and log policy in `configengine`, one new `Spec` field set by the two seed-only modules, two marker accessors; every `lyx config --layer default` write and every hub load changes behavior, standalone and seed-only stacks do not.
