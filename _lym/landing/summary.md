# Drop layout goldens of human-read board output

Output that only a human reads, such as the rendered README and the `--text` listings, no longer has tests that pin its layout or wording.
A format change then no longer forces a large golden rewrite, and the tests stay on behaviour other code relies on.
No production code changes.

## What changes

- `internal/boardengine/render_test.go` loses the README layout goldens.
  It keeps the tests of `RenderToDisk`: rewriting an existing file in place, the output paths and the manifest cleanup.
- `internal/boardengine/view_test.go` loses the README heading and table parsing.
  `TestRenderBoardData` now checks each entry's project, section and `waits_on` against literals, and the section list against a literal.
  `board-data.js` is what `board.html` reads, so that projection stays pinned.
  The render-to-disk subtest also asserts the caller's entry stays flagged as discussed after `RenderToDisk`.
- `internal/boardcli/text_test.go` is deleted, including the `TestRenderCompact` column-layout goldens.
- `internal/boardcli/verbs_test.go` checks `list --text`, `find --text` and `list --text --label` by first line and row handles
  through a new `textListingRows` helper, not byte for byte.
  The sketch check looks for an `S1` row instead of pinning the `sketches:` line and its position.
- `internal/boardengine/layer_test.go` only has a `testtiming:keep` comment reworded, since the render goldens it cited are gone.
- `internal/battencli/readopt_integration_test.go` installs the board replayer with `boardkit.InstallReplay(t)` in the removed-pair readopt subtest.
  Without it the origin reconcile ran with no Bolt replayer registered and the subtest failed independently of the rest of this change.

## Design decisions

- The goldens are deleted, not moved to testdata files.
- No test was added that fails on the base: the change only removes and loosens layout tests, so there is no production defect for one to reach.

## To check by hand

- `list --text` and `find --text` layout checks in `internal/boardcli/verbs_test.go` (`stepListAndFindText`, `stepListShowsASketch`)
  were loosened after a review found them still pinning column layout.
  No reviewer has seen the loosened version; confirm no layout assertion remains there.
- A sweep for other human-read renderers with layout goldens covered `boardengine` and `boardcli`; confirm no other package pins such output.

## Merge danger

**Door:** two-way
**Blast radius:** tests
