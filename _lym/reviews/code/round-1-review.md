---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/boardcli/verbs_test.go:347
    summary: The repository sweep stops short; stepListAndFindText still pins the byte-exact, column-padded layout of lym board list --text and find --text, the human-read output whose unit golden the diff deletes.
    fix: "In internal/boardcli/verbs_test.go, stepListAndFindText: replace the exact-string compares of wantList, wantFind and wantLabelled with checks that do not pin column padding. Keep what the step tests: the prime owner line opens list --text, the P0 root row stands before the first project, note or done entry, the label filter keeps only the bug entry and drops the root row, and find matches only later. Check these by each line's first field (handle) or slug column, e.g. strings.Fields per line, or through the JSON list and find output. In stepListShowsASketch, assert the sketch note is listed (its handle S1 and slug) through the JSON list output or by row field, instead of its position under the sketches: line. Leave the sort, refusal and P0-ordering checks at lines 374-397 as they are."
---

### [MEDIUM] The sweep leaves layout goldens of the compact listing

**Location:** `internal/boardcli/verbs_test.go:347`

**Issue:** Goal point 3 of the amended entry asks to sweep the repository for other renderers of human-read output carrying layout goldens and delete them.
The diff deletes `TestRenderCompact` because the compact listing is read only by people and nothing parses it (commit message; `_lym/landing/summary.md`).
`stepListAndFindText` still compares `list --text`, `find --text` and `list --text --label bug` byte for byte, column padding included, at lines 347–370.
`stepListShowsASketch` pins the `sketches:` line and the row under it at lines 409–411.
No stencil or spec reads `--text`, so these assertions pin layout alone, and the next format change to `RenderCompact` must rewrite them.
The behaviours those steps carry, the root-row placement, the label filter and `find`'s match, stay worth a test and are kept in the fix.
The scope and affected-set lenses and the assistant reported this independently; it is consolidated here.

## Rejected

- code-tests NIT, a done project's `project-<slug>.md` no longer pinned: the file is read only by people, as the lens itself notes, and `isBoardOverview` matches by name pattern without needing the file; the record exempts human-read output from tests.
- Assistant's `internal/boardcli/cli_test.go:1169` `### Features` README assertion: it is a one-substring probe that the hub-declared type reached the render, testing config loading, the same kind of re-render check as the README substring checks in `replay_test.go`, `board_test.go` and `ownerset_integration_test.go`, not a layout golden.

The kept tests still assert what their names state.
`TestRenderToDiskRewritesAnExistingFileInPlace`, `TestRenderToDiskOutputPaths` and `TestRenderToDiskManifestCleanup` are unchanged apart from a removed `BoardDataFile` stat that the loop at `render_test.go:424-431` still covers.
`TestFormatAge` is unchanged.
The literal placements in `view_test.go` cover every entry the README parsing covered, add the section of the done project's task, and keep the entry count at 12.
The section list literal matches the root's sections.
The `Discussed`-flag assertion moved from the deleted README-row test into the `RenderToDisk` subtest of `TestRenderBoardData`.
The `layer_test.go` keep-reason edit is forced by deleting the goldens it named.
The diff changes no production code, carries no trailer, and its `two-way` merge door with a `tests` blast radius holds.
