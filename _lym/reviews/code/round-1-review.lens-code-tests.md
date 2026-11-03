```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour: pinning test**

1. SetValues replaces a mapping-valued P whole from a flow mapping, shaped.
   - Pinned by `TestSetValues` "shaped open map of fixed keys is replaced whole from a flow mapping" (`set_test.go:415`).
   - It also checks that old keys are gone and that tags are kept (`"30"` is absent).
2. SetValues on an unshaped P, and SetValues with `{}`: no SetValues row.
   - The same `setWholeOpenMap` is pinned through SetSparse rows "replaced from a flow mapping" and "emptied by an empty flow mapping" (`layers_test.go:156-157`).
   - Not flagged: the code path is shared.
3. SetSparse replaces P whole.
   - Pinned by `layers_test.go` rows at lines 156-158.
   - The existing-file replacement is checked (`labels:\n a: x` becomes `bug: desc`).
4. A non-mapping value at P is refused naming P and the per-key form.
   - SetValues: `TestSetValues_Refusals` "flow list for a whole mapping open map".
   - SetSparse: row "refuses a value that is no flow mapping".
   - Only `[a]` is tried. A plain string such as `x` takes the same `Kind` check.
   - Malformed braces such as `{a` go through the existing `openMapEntryNode` refusal, which the new tests do not pin at P.
5. A shaped P refuses an unreached key by name, nested included.
   - SetValues: Refusals row "whole shaped open map holding a key its shape lacks" (`build.timeoutt`).
   - SetSparse: row "holding a key its shape lacks" (`tiers.t0.nope`), which exercises the recursion.
   - Module level: `TestConfigSpec_SetWholeSteps` pins `capture holds no key timeoutt` and the unchanged file.
6. A shape-less P accepts any key.
   - Pinned by the `labels` rows in SetSparse and in SetValues Unknown.
7. `unknownKeys` accepts P itself.
   - Pinned by `TestSetValues_UnknownKeys` "an open map lists itself as known" (`labels` accepted, `bogus` unknown).
   - Pinned by the SetSparse rows.
8. `knownKeys` lists P.
   - Pinned by `TestSetValues_UnknownKeys` `wantKnown: "labels"`.
   - Pinned by the `Known` loop in `TestSetSparse`, which now includes `"labels"`.
9. `slices.Compact` in `knownKeys`: none.
10. A dotted P (`parentPath != ""` branch in `setWholeOpenMap`, creating the parent mappings): none.
11. Comment preservation on replace (documented in the `setWholeOpenMap` doc): none.
12. Module wiring, drift `--set capture=` and `compare=`.
    - Pinned by `TestConfigSpec_SetWholeSteps`.
    - It checks `LoadConfig` for both steps, the nested `env` value, and the `dir` default.
    - It does not duplicate `TestConfigSpec_SetNestedKeys`, which covers per-key sets.
13. The `ConfigSpec` doc comment and the `doc.go` prose are static prose and exempt.

**Findings**

- **F-T1** (LOW, scope), `internal/yamlengine/set.go:387-393`. The `parentPath != ""` branch of `setWholeOpenMap` has no row. Declared open maps with dotted paths exist (`buttons.task`, `buttons.beam`, `buttons.prime` in `internal/reedengine/config.go`). Fix: add a SetValues and SetSparse row with a dotted open map P holding a mapping, where the parent is absent from `existing`. Evidence: `set.go:387-393`; no dotted `openMaps` in the new rows (`set_test.go:415-424`, `layers_test.go:156-160`).
- **F-T2** (LOW, scope), `internal/yamlengine/set.go:369`. The doc says the old value's comments are kept, but no assertion checks it. The drift template carries a line comment on `capture: {}`. Fix: add a SetValues row whose template has `build: {}  # note` and assert the comment survives the whole-map set. Evidence: the `contains` and `notContains` of the new rows hold no comment text.
- **F-T3** (NIT, scope), `internal/yamlengine/layers.go:302`. `slices.Compact` in `knownKeys` has no row proving the dedup. The `Known` loop only uses `Contains`. Fix: assert in `TestSetSparse` or the Known test that `labels` appears exactly once, using a template where P is also a leaf path. If P can never be both, drop `Compact` instead.

**Tool calls:** none. My context already held every file and the whole diff.
