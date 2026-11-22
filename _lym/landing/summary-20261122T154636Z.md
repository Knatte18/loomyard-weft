# Drop the layout goldens of human-read board output

The board README, project files and design docs are read only by people.
Their layout and wording now carry no test, so a format change no longer drags a matching golden rewrite along with it.

## What changed

- `internal/boardengine/render_test.go` loses every README, project-file and design-doc golden and the substring checks on rendered rows.
  It keeps the tests of behaviour other code relies on:
  `RenderToDisk` rewriting a file in place, its accepted and refused output paths, and the manifest cleanup.
  `TestFormatAge` stays, since `lym board list --text` and the README both format ages through `FormatAge`.
- `internal/boardengine/view_test.go` no longer parses README headings and tables to find where an entry stands.
  `TestRenderBoardData` now pins each `board-data.js` entry's project, section and waits-on handles,
  and the section list, as literals from the fixture.
  `board.html` reads these values, so they keep their test.
- The `RenderToDisk` subtest of `TestRenderBoardData` now also asserts that a render leaves the caller's `Discussed` flag set.
  That assertion lived in a deleted README-row test.
- The repository sweep found one more layout golden of human-read output:
  `internal/boardcli/text_test.go`'s `TestRenderCompact`, which pinned the column layout of `lym board list --text`.
  Nothing parses that listing, so the file is deleted.
  Other golden-style tests the sweep met pin text an agent or parser reads, such as refusal messages, halt notes and the settings document, and stay.

## Design decisions

- `TestFormatAge` is kept: it pins the unit boundaries of a shared formatter rather than a document's layout.
- No renderer code changes; only tests are deleted or rewritten.
- The entry is labelled `bug`, but no test can fail on the merge base, since the change fixes no production defect.
  The decision record's "No test seam" entry records this.

## Merge danger

**Door:** two-way
**Blast radius:** tests

The run `readme-declutter` edits `render_test.go` and `view_test.go` in parallel.
Whichever lands second takes a merge conflict in those files, settled by keeping this change's deletions.
