---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/boardengine/render.go:419
    summary: settingValue uses json.Marshal, which HTML-escapes angle brackets and ampersands in non-string values, so a list holding such a character renders as a unicode escape
    fix: In settingValue, encode with a json.Encoder whose SetEscapeHTML(false) is set, writing to a strings.Builder, and trim the trailing newline, instead of calling json.Marshal.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: design
    location: internal/boardengine/render.go:378
    summary: A setting value that contains a backtick ends the code span early and garbles the rest of the cell
    fix: In entryCell, wrap each setting line in a code span fenced by a backtick run longer than the longest backtick run in the line, padded with a space on each side when the line starts or ends with a backtick.
    origin: lens:code-correctness
---

### [LOW] HTML escaping in non-string setting values

**Location:** internal/boardengine/render.go:419
**Issue:** `json.Marshal` rewrites `<`, `>` and `&` inside a list or other non-string value as `<`, `>` and `&`. A setting such as `["a&b"]` therefore renders as `["a&b"]`, which is not the compact JSON the `settingValue` comment promises. String values are unaffected.

### [LOW] Backtick in a setting value breaks the code span

**Location:** internal/boardengine/render.go:378
**Issue:** The change description accepts this as unlikely. A backtick in a value closes the span early, so the rest of the cell renders as plain markdown. The table row itself stays intact, because `cellText` still escapes pipes and collapses line breaks.

## Rejected

- Settings on a Notes or Done entry having no test: the summary names the shared cell as a design decision, and the `base` row already pins the shared code path, so a second row would duplicate it.
- An empty module mapping such as `{"darn": {}}` producing no line: `task.go:270` normalises an empty `run_settings` to nil, and there is nothing to show.
- No test-budget edit needed: the diff adds no top-level test, so the `internal/boardengine` row is unaffected.
