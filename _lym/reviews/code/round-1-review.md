---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: handler
    location: internal/driftengine/config.go:86
    summary: "drift declares capture and compare as open maps decoding into stepFile (command, dir, env, timeout_min), but declares no shape, so `lym config drift --set capture.env.X=1` still lands one level too high, the bug the Idea names for any module"
    fix: "Give drift's ConfigSpec an OpenMapShapes entry for each of the two open maps (stepFile is the decode type of its capture and compare keys, use the constants stepCapture and stepCompare as the map keys), built with yamlengine.ShapeOf(stepFile{}) from the type driftengine already decodes. Add a driftengine test that sets capture.timeout_min and capture.env.X through configengine.Set and loads the file with LoadConfig, and that a misspelled key under capture is refused naming it. Extend the drift template comment and the configcli doc.go sentence only as far as the behavior changes."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/yamlengine/set_test.go:350
    summary: "The documented whole-entry form `--set tiers.tierN=...` and a flow list on a single-value field now run through the new setShapedValue Fields, Entries and brace/bracket branches, and no TestSetValues or TestSetSparse row sets such a key under a shaped open map"
    fix: "Add one TestSetValues row and one TestSetSparse row that set a one-segment key under a shaped open map with a flow mapping value (tiers.t0 with command x) and assert the entry lands whole. Add one TestSetValues row that sets a bracketed value on a single-value field (add a covers []int field to shapedTier) and assert it lands as a flow list."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/yamlengine/set.go:324
    summary: ShapeOf documents an untagged field keyed by its lower-cased name, a tag option stripped from the key, and unexported fields left out, and the shapedTier test type exercises none of the three
    fix: "Add to shapedTier an untagged exported field, a field tagged with an omitempty option and an unexported field, then add a TestSetValues row that sets the first two and a TestSetValues_Refusals row that names the third as a key the entry lacks."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/yamlengine/set.go:201
    summary: the up-front shapedKeyError check that makes a multi-pair set all-or-nothing has no row that sends a valid pair followed by a refused one
    fix: "Add a TestSetValues_Refusals case, or a small separate test, that calls SetValues with a valid shaped pair first and a misspelled shaped key second and asserts the error and that no merged bytes are returned. The table takes one pair today, so give it a pairs slice or add the separate test."
---

### [MEDIUM] Drift's capture and compare open maps keep the one-level-too-high bug

**Location:** internal/driftengine/config.go:86
**Issue:** The Idea asks that `--set` take a dotted path to any key the module's schema reaches, nested maps included.
The diff declares shapes for gate's `tiers` and `build` only.
Drift's `ConfigOpenMaps` returns `capture` and `compare`, each decoded into `stepFile` with `command`, `dir`, `env` and `timeout_min` (`internal/driftengine/config.go:42-48`), the same shape as gate's build step.
For them `capture.env.X=1` still writes an entry literally named `env.X`, and `capture.timeout_min=30` writes a string where an int is decoded.
The shape mechanism is opt-in per open map, so the gap is a missing declaration, not a mechanism defect.

### [MEDIUM] The whole-entry and bracketed-value branches of the shaped write have no test row

**Location:** internal/yamlengine/set_test.go:350
**Issue:** `setShapedValue` now carries `--set tiers.tierN={...}`, the form `internal/configcli/doc.go:53` documents, through its `Fields`/`Entries` and brace/bracket branch, and a bracketed value on a single-value field such as a list.
Every new row uses a key two or more segments deep, so neither branch fails a test if it breaks.

### [LOW] ShapeOf's tag rules are only partly exercised

**Location:** internal/yamlengine/set.go:324
**Issue:** The doc comment promises an untagged field keyed by its lower-cased name and a tag's options dropped from the key.
`shapedTier` has only plain tagged fields, one `-` field, and the pointer, map, string and `yaml.Node` types.
The unexported-field skip is also unpinned.

### [LOW] The all-or-nothing check on a multi-pair set is untested

**Location:** internal/yamlengine/set.go:201
**Issue:** `shapedKeyError` runs before any pair applies so one bad shaped key rejects the whole call, as the `SetValues` doc says.
`TestSetValues_Refusals` takes one pair per case, and `TestConfigSpec_SetNestedKeys` sends one pair in its refusal step, so a regression that validated each pair only as it applied would pass.

## Rejected

- Lens code-correctness: `ShapeOf` ignores `yaml:",inline"` and has no cycle guard. Neither occurs in `tierFile`, `buildFile` or any declared shape, and the rubric's code-quality rule bars handling a scenario nothing produces.
- Lens code-affected-set reported no finding; the handler found no set member left wrong in its own reading either, beyond F1.
