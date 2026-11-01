```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list**

- **A key inside a shaped entry, mappings created on the way** (`tiers.t0.env.NAME`, `timeout_min`).
  - Pinned by `TestSetValues` row "shaped open map takes one key inside an entry…", which also runs the idempotence second pass.
  - Pinned by `TestSetSparse` rows "a key inside a shaped open-map entry…" and "…flow entry keeps the entry's keys".
  - Pinned by `TestConfigSpec_SetNestedKeys` end to end.
- **A number is written as a number and a text field as a string** (`timeout_min`, `command=42`). Pinned by the `TestSetValues` fixed-keys row and the `gateslot` test, which loads the file back.
- **A `yaml.Node` field writes the scalar under its own tag** (`slots=all`). Pinned by the `TestSetValues` fixed-keys row.
- **An unshaped open map keeps a dotted rest as one entry name.** Pinned by the `TestSetValues` "unshaped open map…" row. The `SetSparse` path has no such row, though every `SetSparse` case passes a `shapedOpenMaps` map in which only `tiers` and `build` are shaped. `labels` is unshaped in that map and the pre-existing `labels` rows still run, so that gap is covered.
- **Refusals** (`shapedOpenMapKey`). Pinned by the `TestSetValues_Refusals` rows: missing key, `-`-tagged key, below a single value, empty segment, and entry holds a scalar. `SetSparse` has one refusal row, for the missing key. The code is shared, so that is enough.
- **Gate declares shapes for `tiers` and `build`.** Pinned by `TestConfigSpec_SetNestedKeys`, including the refusal that names `envv` and leaves the file byte for byte.
- **`Spec.OpenMapShapes` passed through both `Set` paths** (`SetValues` for standalone/seed-only, `SetSparse` for sparse hub). Pinned by the `gateslot` test for whichever path `seedGateConfig` takes. I did not read `seedGateConfig`, so I cannot say the other path is pinned (see F1).
- **`ShapeOf` derivation.** Pinned by `shapedTier`: yaml tag, `-` skipped, pointer, map, string, `yaml.Node`. Not pinned: the untagged-field lower-cased name, a tag with options (`name,omitempty`), unexported fields skipped, and a pointer-to-struct top-level argument (see F3).
- **One-segment key under a shaped open map writes the whole entry from a flow value** (`tiers.tier0={...}`, `build={...}`). Pinned by none. The new shaped path now carries it, since the `Fields`/`Entries` case and the `{`/`[` case go through `setShapedValue`. All new rows use dotted keys of two or more segments after the map. `watchdogline_test.go:60,63` only checks help text. The new `TestConfigSpec_SetNestedKeys` doesn't set `tiers.tier0={...}`, and `TestSetSparse` has no such row either.
- **A flow `[`/`{` value for a single-value leaf under a shaped map** (for example a `covers` list). Pinned by none (see F2).
- **A failed set writes nothing when a later pair is bad.** Partly pinned. The `gateslot` test checks the single-pair refusal leaves the file unchanged. No row passes a valid pair followed by a refused one (see F4).
- **Help text, doc comments and template comment.** Static prose, exempt.

**Findings**

- **F1, MEDIUM, design.** `internal/yamlengine/set_test.go:350-376`, `internal/yamlengine/layers_test.go:149-152`. The documented pre-existing form `--set 'tiers.tierN={...}'` (`internal/configcli/doc.go:53`) now goes through `setShapedValue`'s `Fields`/`Entries` branch for gate. No test row sets a one-segment key under a shaped open map in `SetValues` or `SetSparse`. Evidence: every new shaped row uses a key with two or more segments after the map name, and my grep found only help-text assertions for `tiers.tierN={...}`. Fix: add one `TestSetValues` row and one `TestSetSparse` row for `tiers.t0={command: x}` under shapes, asserting the flow mapping lands whole.
- **F2, LOW, design.** `internal/yamlengine/set.go:460`. The `{` and `[` branch for a single-value leaf (for example `tiers.t0.covers=[1,2]`) has no row. Fix: add a `TestSetValues` row with a `[`-valued key on a single-value field, or on a `[]int` field in `shapedTier`.
- **F3, LOW, design.** `internal/yamlengine/set.go:324-354`. `ShapeOf` has no direct row for an untagged field, a tag with `,omitempty` options, an unexported field, or a pointer-to-struct argument. Fix: add an exported-field/untagged/options field to `shapedTier`, plus a refusal or acceptance row for it.
- **F4, LOW, design.** `internal/yamlengine/set.go:201,385`. The pre-validation that makes a multi-pair call all-or-nothing is untested. Fix: add a `TestSetValues_Refusals` row that sends a valid pair then a bad shaped key and asserts nothing is merged. This needs a multi-pair variant of the table, which currently takes a single `pair`.

**Tool calls**
1. `grep` over `*_test.go` for `tiers.tierN=`, `build={` and `seedGateConfig`. The manifest lacked other tests of the flow-entry write form.
