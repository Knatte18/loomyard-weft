---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/yamlengine/set.go:201
    summary: >-
      A mapping-declared open map holding a list now takes a string or list at P itself, so lyx config board --set labels=x overwrites a list-shaped labels with the string x
    fix: >-
      In isScalarOrListOpenMap (internal/yamlengine/set.go), decide the shape from the template's own value for P, not the value after carryOpenMaps.
      SetValues and SetSparse both carry the existing file's value into templateNode before the per-pair loop, so a list-shaped labels in the file makes the check true for a mapping-declared open map.
      Capture each open-map path's template Kind before carryOpenMaps runs, in both functions, and pass that to isScalarOrListOpenMap, so only a path whose template value is a scalar or a list takes the new branch.
      Add a row to TestSetValues_Refusals and one to TestSetSparse where the template holds labels as an empty mapping, the existing file holds labels as the list [a], and the pair is labels=x.
      It must stay refused as before, with the file unchanged.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/yamlengine/set.go:213
    summary: >-
      A bracketed value with trailing text after the closing bracket is written as the list alone and the text is dropped silently, though the entry says an unwritable value is refused
    fix: >-
      In scalarOrListNode (internal/yamlengine/set.go), reject a value whose YAML does not end at the list's closing bracket.
      After replaceSequence succeeds, require that strings.TrimSpace(pair.Value) ends with a closing square bracket, and otherwise return refusal.
      A value such as ["a"] trailing then returns the refusal error and nothing is written.
      Add the value as a row to TestSetValues_Refusals, and to TestSetSparse with wantErr.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/yamlengine/layers_test.go:118
    summary: >-
      The change description claims the empty-list and nested-list rows exist on both write paths, but TestSetSparse has no such rows
    fix: >-
      Either add an empty-list row (value [], want rounds: [] and a newline) and a nested-list refusal row (value [[a], b], wantErr) to TestSetSparse in internal/yamlengine/layers_test.go, or correct the Tests paragraph of _lyx/landing/summary.md to say the empty-list and nested-list rows are on the full write path only.
      Prefer the correction when the shared scalarOrListNode path is already covered by the SetValues rows, to avoid redundant tests.
---

### [BLOCKING] A list-shaped mapping open map is overwritten by --set P=value

**Location:** `internal/yamlengine/set.go:201`, `internal/yamlengine/set.go:125`, `internal/yamlengine/layers.go:197`
**Issue:** `isScalarOrListOpenMap` asks whether the value at P in the working tree is a non-mapping.
Both write paths call `carryOpenMaps` first, which replaces the template node at P with the file's node.
With a template of `labels: {}` and a file holding `labels: [a]` (the list-shaped state the old help text and the package doc warn about), I ran `SetValues` and `SetSparse` with `labels=x`.
Both write `labels: x` and return no error; `labels=[b]` writes `labels: [b]`.
Before the diff, `labels` was a list key at that point and a scalar value was refused (the `list key refuses a scalar` row), and a list-shaped map was to be rewritten in an editor.
The change description states that mapping-valued open maps such as board's `labels` are unaffected and that `P` itself stays outside the set path.
For a mapping-declared map this does not hold once the file's value is not a mapping, and the help text that warned about the case was removed in the same diff.
The new branch also runs before the `splitOpenMapKey` branch, so ordering does not guard it.
The other focus points hold: comments on the replaced node are kept in both paths (`value.HeadComment` copy in `SetValues`, `setEntry` in `SetSparse`), and `P.<name>` entries are unchanged.

### [MEDIUM] Text after the closing bracket is dropped without a refusal

**Location:** `internal/yamlengine/set.go:213`
**Issue:** `scalarOrListNode` relies on `replaceSequence`, which unmarshals the value and accepts it when the document's single node is a sequence.
I ran `--set` with the value `["a"] trailing`: both paths return no error and write `plan_review: ["a"]`, losing `trailing`.
The board entry says a value that cannot be written in the key's shape is refused, never written wrong, and the change description says a bracketed value that does not parse as a list of scalars is refused with nothing written.
The value is a bracketed non-list-of-scalars here, yet it is accepted in a changed form.

### [LOW] Change description overstates sparse-path test rows

**Location:** `internal/yamlengine/layers_test.go:118`
**Issue:** The summary's Tests paragraph says the empty list and the nested-list refusal rows are on "both the full and the sparse write paths".
`TestSetSparse` gained three rows only: quoted flow list, list key taking a scalar, and the unquoted bracketed refusal.
`TestSetValues` has the empty list and the nested-list refusal.
The claim does not match the diff.
