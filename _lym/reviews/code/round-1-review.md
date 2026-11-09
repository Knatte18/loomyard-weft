---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/configcli/configcli_test.go:1170
    summary: >-
      no CLI-level test drives a dotted key through lym config profiles --set, the board entry's own example,
      so the post-write load check and rollback are unpinned for it
    fix: >-
      In TestDispatchSetProfiles add one row with setFlag `light.loom.loom.plan_max_bounces=2` against the seeded file,
      wanting the hub file to hold the profile with that one value changed and the rest kept.
      Add a second row with a dotted key that leaves the profile in a wrong shape, such as `light.loom.loom.no_such_key=1`,
      wanting code 1, the file as seeded and the profile load refusal in the error.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/yamlengine/set_test.go:906
    summary: >-
      TestSetEntries has no row for several branches of the dotted path,
      namely a bracketed value, a malformed flow value, a non-number scalar tag, a null parent and a replaced leaf's comments
    fix: >-
      In TestSetEntries add rows for these, each pinning an exact output or error.
      Key `docs.darn.fan` with value `[a, b]` wants `fan: [a, b]`.
      Key `docs.darn` with value `{a: 1` wants an error naming `docs.darn` and containing `double-quote`.
      Key `docs.darn.on` with value `true` and key `docs.darn.x` with value `null` want an unquoted `true` and `null`.
      Key `docs.darn.writer` over a null `darn` wants a mapping created.
      Key `docs.darn.writer` over a leaf carrying a line comment wants the comment kept.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/configengine/layers.go:78
    summary: >-
      the Spec.JSONEntries field comment still says a set writes each pair as one whole top-level entry,
      which is false for a dotted key
    fix: >-
      In the Spec doc comment at layers.go:78 reword the JSONEntries sentence so it says a set writes a pair with an undotted key
      as one whole top-level entry from a JSON object, and a pair with a dotted key as the one value below an entry,
      with no template key check.
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-correctness
    location: internal/yamlengine/set.go:862
    summary: >-
      any dot in a key is now a path, so a profile name with a dot, which runprofile accepts,
      can no longer be added or replaced whole by --set, and nothing documents or refuses the case
    fix: >-
      Say it in the user-facing text.
      In internal/configcli/doc.go after the dotted-key paragraph, and in the SetEntries doc comment,
      add one sentence that a dot in a key always separates levels, so a top-level name holding a dot cannot be set by --set.
      No code change is needed, since the board entry licenses the dotted reading.
---

### [LOW] No CLI-level test of a dotted profile set

**Location:** internal/configcli/configcli_test.go:1170
**Issue:** The board entry's example, `docs.darn.loom.code_max_bounces=0`, runs through `configcli` → `configengine.Set` → `yamlengine.SetEntries` and then the hub-wide load check that rolls the file back on a refusal.
Every `TestDispatchSetProfiles` row is a whole-entry key, so the `SetEntries` rows are the only coverage.
The change description's claim that a dotted key leaving a profile in the wrong shape is refused and the file restored has no test.

### [LOW] Unpinned branches of the dotted path in TestSetEntries

**Location:** internal/yamlengine/set_test.go:906
**Issue:** The six new rows each pin a distinct behavior and none duplicates another.
`setEntryValue` also has a flow-list case, a refusal of a malformed flow value, scalar tags other than a number, a null value on the way and the comment-keeping of a replaced leaf, and no row fails if any of them breaks.
The `SetEntries` doc comment promises the comment keeping.

### [LOW] Stale JSONEntries field comment

**Location:** internal/configengine/layers.go:78
**Issue:** The diff updates the prose of `Set`, `setEntries` and the package doc, but the field's own comment still says "each pair as one whole top-level entry from a JSON object".

### [LOW] A dotted top-level name can no longer be set

**Location:** internal/yamlengine/set.go:862
**Issue:** `runprofile` rejects only an empty profile name, so `v1.5` was a legal profile name that `--set 'v1.5={...}'` could add.
It is now read as the path `v1` → `5`.
The board entry asks for the dotted reading, so the behavior is licensed; the gap is that no doc says a dot in a key always separates levels.

## Rejected

- code-correctness F2 (NIT): a dotted scalar cannot spell the string `"0"` or `true`. The doc comment states the rule (the tag its own text resolves to), the board entry does not ask for string forcing, and `SetValues` leaf sets share the limit.
- code-correctness note on `" 5 "` marshalling: the lens did not run it and reports the same pattern in `setShapedValue`, so it is no evidence of a defect in this diff.
- code-tests T1, T2 and T3 and code-tests T4 as separate findings: merged into F2 and F1, since they ask for rows in the same two tests. code-affected-set F3 is merged into F1 for the same reason.
- code-affected-set note on `internal/runprofile/template.yaml:14`: the change description says it was left alone on purpose, and no rule asks for a seeded-file comment change.
