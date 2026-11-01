```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** I traced each changed function and branch with a concrete input. No BLOCKING or MEDIUM defect found.

**Cases run**
- `ShapeOf(map[string]tierFile{})`: gives `Entries → Fields{about, command, …, env→Entries(Text), slots→{}, covers→{}, timeout_min→{}}`. Correct. Pointer, `*int`, `*bool` and `yaml.Node` fields reduce to a single-value shape. `yaml:"-"` fields are dropped. An untagged field is keyed by its lower-cased name, as yaml.v3 does.
- `shapedOpenMapKey("tiers.tier0.env.LYM_TEST_TIER")`: walks Entries, then Fields, then Entries, and returns the Text shape. Correct.
- `shapedOpenMapKey("build.timeout_min")`: returns a single-value shape. Correct.
- `shapedOpenMapKey("tiers.tier0.envv.X")`: errors "tiers.tier0 holds no key envv; it holds …". `walked` is right.
- `shapedOpenMapKey("tiers.tier0.command.x")`: errors "holds one value". Correct.
- `shapedOpenMapKey("tiers.tier0..x")`: errors on the empty segment. Correct.
- `shapedOpenMapKey("tiers.tier0.env.")`: errors on the empty segment. Correct.
- `shapedOpenMapKey("tiers.")`: `splitOpenMapKey` rejects an empty rest, so it is unknown. The shape walk is never entered.
- `shapedOpenMapKey("tiers.tier0")`: gives the `tierFile` Fields shape. `setShapedValue` then takes the flow-mapping path through `openMapEntryNode`, the same as the old whole-entry write.
- `shapedOpenMapKey("tiers.tier0.env.A.B")`: errors "tiers.tier0.env.A holds one value". An env name containing a dot is refused. That is acceptable, but the message does not say why.
- `shapedOpenMapKey` with `shapes == nil`: the map lookup fails, so it returns `(nil, false, nil)`. This is the unshaped path and `TestSetValues_UnknownKeys` passes `nil`. Correct.
- `setShapedValue`, value `5` on a `*int` field: `scalarSetTag("!!null","5")` resolves to `!!int`, so it is written as a number. Correct.
- `setShapedValue`, `abc` on `*int`: written as `!!str`. The loader refuses it later and `setChecked` rolls back (`configcli.go:283`).
- `setShapedValue`, `~` on `*int`: written as `!!null`. Correct.
- `setShapedValue`, empty value on a non-text field: written as an empty null scalar. Correct.
- `setShapedValue`, `a: b` on a non-text field: the parse gives a mapping, so the tag falls back to `!!str`. Correct.
- `setShapedValue`, Text field with `{x}` or `0`: written as a string, because the `Text` case comes first. Correct.
- `setShapedValue`, `covers=[1,2]`: single-value shape with a `[` prefix, so it goes through `openMapEntryNode` as a flow list. Correct.
- `setShapedValue`, `tier0` holding a scalar: `mappingAt` errors and the wrap adds "rewrite it as a map to set …". Correct.
- `setShapedValue`, `tier0` null: `mappingAt` converts it to a mapping. Correct.
- `setShapedValue`, empty `tiers: {}` in the template: `mappingAt` resets `Style` to 0, so it is written as a block. Correct.
- `setShapedValue`, flow-style tier entry: the key is appended inside the flow mapping. This is pinned by the "keeps the entry's keys" row.
- `SetValues` failure paths: `shapedKeyError` runs before any mutation, after `unknownKeys`. `setShapedValue` failing mid-loop leaves only in-memory nodes changed, never a write. `Set` returns the error and calls `removeIfScaffolded`. Correct.
- `SetSparse` failure paths: same validate-then-write order. `configengine.setSparse` returns before writing (`configengine/set.go:166-169`). Correct.
- Caller path: `configcli.setChecked` calls `configengine.Set(dirs, module.Spec(), …)` (`configcli.go:279`). So the gate spec's `OpenMapShapes` reaches `Set`. Errors return unwrapped to `output.Err`. A write that breaks the load is rolled back. Correct.
- Unshaped open maps (`tier_parallel`, board `labels`): the lookup fails and the old `setOpenMapEntry` path keeps a dotted rest as one name. Correct.
- Comments on a replaced entry: `setEntry` keeps the old node's comments. The old `setOpenMapEntry` dropped them, so this is a harmless difference.

**Findings**

- **LOW, design.** `internal/yamlengine/set.go:80-109` (`shapeOfType`). `ShapeOf` does not model yaml.v3's `,inline` option, and it recurses with no cycle guard.
  - A field tagged `yaml:",inline"` is keyed by its lower-cased name instead of being flattened. It would refuse valid keys and accept wrong ones.
  - A self-referential type (a struct holding `map[string]Self` or `*Self`) recurses forever.
  - Neither occurs in `tierFile` or `buildFile`, so nothing is wrong today.
  - Evidence: the `shapeOfType` struct case reads only the `yaml` tag name and ignores the options after the comma. The doc comment does not say `inline` is unsupported.

**Tool calls**
1. `grep` of `internal/configcli/configcli.go` for `Set(`, `Spec()` and `OpenMaps`: to check that `configcli` passes the gate spec's shapes to `Set`. Neither the manifest nor the diff showed this.
2. `Read` of `internal/configcli/configcli.go:255-304`: to confirm the `setChecked` rollback and that `Set` errors reach the operator unchanged. Neither the manifest nor the diff showed it.
