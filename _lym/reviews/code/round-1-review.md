---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/yamlengine/set.go:401
    summary: "A whole-map set checks only which keys a shape reaches, so a mapping or list under a text field (capture={command: {a: b}}) is written, where capture.command=... always writes a string."
    fix: "In mappingShapeError (internal/yamlengine/set.go), when shape.Text is set and the node at that key is not a scalar, return an error naming path.name and saying the field takes one string; keep returning nil for a non-text single value. Add a row to TestSetValues_Refusals and one to TestSetSparse for capture-style {command: {a: b}} under a shape whose command field is Text."
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/yamlengine/set.go:221
    summary: "The whole-open-map branch runs before the shaped-key branch in SetValues but after it in SetSparse, so a path that is both a declared open map and under another shaped open map would route differently on the two write paths."
    fix: "Put the slices.Contains(openMaps, pair.Key) branch in the same position relative to the shapedOpenMapKey branch in SetValues (internal/yamlengine/set.go:221) and SetSparse (internal/yamlengine/layers.go:258); moving the SetSparse branch above the shapedOpenMapKey check at layers.go:202 matches SetValues."
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/yamlengine/set.go:387
    summary: "The parentPath branch of setWholeOpenMap, which creates missing parent mappings for a dotted open map holding a mapping, has no test row."
    fix: "Add one row to TestSetValues and one to TestSetSparse (set_test.go, layers_test.go) with a dotted open map such as a.b declared in openMaps, a mapping-valued pair at a.b and an existing document lacking a, asserting the merged output holds a.b with the set keys."
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/yamlengine/set.go:370
    summary: "setWholeOpenMap is documented to keep the replaced value's comments, but no test asserts a comment survives the replacement."
    fix: "Add a TestSetValues row whose template has a line comment on the open map (for example build: {}  # note) and a whole-map pair on it, and assert the output still contains the comment."
  - id: F5
    severity: NIT
    class: scope
    origin: lens:code-tests
    location: internal/yamlengine/layers.go:302
    summary: "slices.Compact in knownKeys removes the duplicate an empty-mapping open map leaf makes, but no test asserts the key is listed once."
    fix: "In TestSetSparse (internal/yamlengine/layers_test.go), assert that got.Known holds labels exactly once, using sparseTemplate where labels is an empty mapping; or drop slices.Compact if the duplicate cannot occur."
---

### [LOW] Whole-map shape check ignores the kind of a value under a text field

**Location:** internal/yamlengine/set.go:401
**Issue:** `mappingShapeError` returns nil when the shape has neither `Fields` nor `Entries`, so `capture={command: {a: b}}` or `{command: [x]}` is written into a string field.
The per-key form `capture.command=...` writes a string through `setShapedValue`'s `Text` case, so the whole-map form is weaker than the form it sits beside.
The bad file is caught only by the next `LoadConfig` or the hub-wide validate-before-commit check.
The decision record's Goal asks only that the whole-map form be accepted, so this does not block it.

### [LOW] Whole-map branch ordering differs between SetValues and SetSparse

**Location:** internal/yamlengine/set.go:221
**Issue:** `SetValues` tests `slices.Contains(openMaps, pair.Key)` before `shapedOpenMapKey`, and `SetSparse` tests it after.
No module declares nested open maps whose paths overlap (reed's `buttons.*` hold lists and take the scalar-or-list branch first), so no input reaches the difference today.

### [LOW] Untested parent-creation branch of setWholeOpenMap

**Location:** internal/yamlengine/set.go:387
**Issue:** The `parentPath != ""` branch calls `mappingAt` to create or walk parent mappings for a dotted open map.
None of the new rows in `set_test.go` or `layers_test.go` declares a dotted open map holding a mapping.

### [LOW] Comment preservation on replace is untested

**Location:** internal/yamlengine/set.go:370
**Issue:** The doc comment states the old value's comments are kept, through `setEntry`.
The drift template carries a line comment on `capture: {}`, so this is the shipped case, yet no assertion checks it.

### [NIT] slices.Compact in knownKeys has no row proving it

**Location:** internal/yamlengine/layers.go:302
**Issue:** An open map whose template value is `{}` is also a leaf path, so `knownKeys` would list it twice without `slices.Compact`.
The `Known` assertions use `slices.Contains` and cannot see a duplicate.

## Rejected

- Whole-map `--set` now applies to every module's mapping-valued open map (board `labels`, gate `tiers`, and the rest), beyond drift's `capture` and `compare`: the fix sits in the shared engine that refused the key, the commit message states "every declared open map", and the decision record's Goal names no module-specific route, so it is not a scope finding.
- The Goal's alternative, a key description naming the per-key form: not needed, since the template's own description already prescribes `capture={...}` and the diff makes it work; the `ConfigSpec` comment change in `internal/driftengine/config.go` matches the new behavior.
- Redundant `mappingAt` call in `SetSparse` before the whole-map branch: harmless because `mappingAt` is idempotent, and not a defect.
- `TestConfigSpec_SetWholeSteps` overlaps the yamlengine rows on the shape refusal: it alone pins the module wiring and the unchanged file after a refusal, so it covers behavior no other test covers.
