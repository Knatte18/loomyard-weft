```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over `runFieldLines` and the changed `entryCell` loop.

**Per function and branch, the case I ran and the result**

- **`runFieldLines`, both fields empty** (most fixture rows, such as T2): `lines` stays nil and the function returns nil. `append(nil, settingLines(...)...)` gives the settings alone, so the cell is unchanged from before. Correct.
- **`runFieldLines`, only `PullRequest` set** (T3, `"on"`): returns `["pull_request: on"]`, and the `OnLand` branch is skipped. Correct.
- **`runFieldLines`, both set** (T1, `merge-in` and `refresh`): returns `["pull_request: merge-in", "on_land: refresh"]` in that fixed order. Correct.
- **`runFieldLines`, only `OnLand` set** (a front task with no override): returns `["on_land: <key>"]`. Correct.
- **`entryCell` loop, run fields with nil `RunSettings`**: `settingLines(nil)` ranges over a nil map and sorts nil, which yields nil. The append returns the run-field slice, and one bullet is written per field. Correct.
- **`entryCell` loop, both slices empty**: the loop does not run, and the cell is the title plus the brief. Correct.
- **`entryCell` loop, aliasing**: `runFieldLines` builds a fresh slice on each call, so appending into its spare capacity cannot change any other caller's data. Correct.
- **`entryCell`, value with a pipe, a backtick or whitespace**:
  - `cellText` escapes `|` as `\|` and collapses whitespace runs to one space.
  - `codeSpan` picks a fence longer than any backtick run and pads the text when it starts or ends with a backtick.
  - The row stays one table row. This is the same path the run settings already take.
  - The values are also validated upstream: `pull_request` must be on, off or merge-in, per `internal/boardengine/doc.go:404-405`, and `on_land` must be a key in the action table, per `internal/fronttask/fronttask.go:181`.
  - Correct.
- **Error paths**: neither function returns an error or calls anything that can fail, so there is no error path to check.

**Findings:** none.

**Tool calls:** none. My inherited context already held `render.go:858-889`, `codeSpan`, `cellText`, `settingLines` and the validation sites.
