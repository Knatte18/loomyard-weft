```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that would fail if it broke.

Each behaviour and the test that pins it:

- **`pull_request` bullet renders as `pull_request: <value>`.** Pinned by `TestRenderReadmeGolden`, "tasks, notes, done and abandoned entries". Row T3 expects `` • `pull_request: on` `` and row T1 expects `` • `pull_request: merge-in` `` (`internal/boardengine/render_test.go:377`, `:385`).
- **`on_land` bullet renders as `on_land: <key>`.** Pinned by the same row T1, `` • `on_land: refresh` `` (`render_test.go:385`).
- **`pull_request` comes before `on_land`.** Pinned by row T1, the only fixture task that sets both (`render_test.go:385`).
- **Run-field bullets come after the brief and before the sorted run settings.** Row T1 pins brief, then fields, then settings. Row T3 pins fields before settings when there is no brief (`render_test.go:377`, `:385`).
- **A field left empty adds no bullet.** Pinned by the unchanged golden rows: T2, whose entry cell is unchanged and has no field bullet, and every note and done row. Any stray bullet would break those rows (`render_test.go:254`, and the unchanged rows of the same `want`).
- **The code-span and pipe-escape wrapping of each line.** Unchanged code (`codeSpan(cellText(line))`), already pinned for settings by row T2's escaped `opus\|fast`. Run-field values come from validated sets (`on`/`off`/`merge-in`, and the `fronttask` action keys), so they cannot hold a pipe or a backtick. No new case is needed.
- **`entryCell` in the Running, Ready, layer, Draft, Notes and Done tables.** The same function feeds every table, and the golden covers the Draft and Independent tables, so no per-table row is needed.

Test economy: the diff adds no new test function. It adds fixture fields and golden-row changes to the one existing test, which the redundancy report marks unique. The two added comment lines in the test doc describe what rows T1 and T3 assert. They are not assertions on static prose.

Findings: none.

Tool calls: none.
