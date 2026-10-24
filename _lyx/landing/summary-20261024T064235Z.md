# Record-batch: accept a Go test written in its directory's other package

`lyx webster record-batch` failed a batch with blocking `create-not-done` findings when a Create target `<dir>#TestX` was written into the external test package `<dir>_test`.
The finding itself named the declaration it had found (`TestX is declared as <dir>_test#TestX`), and only `lyx webster recover-batch` got past it.
A plan cannot know at plan time which package a new test file declares, so record time is where this is resolved.
Fixes #851.

## What changes

- `planparser.GoPackageCounterpart` maps a Go member glyph to the same member in its directory's other package: `<dir>` to `<dir>_test`, and `<dir>_test` to `<dir>`.
  It reports false under a non-go plan, for a self glyph and for a file-unit glyph.
- `planglyph.DoneChecks`: under a go plan, a Create target or Rename new side answered not_found is done when the worktree's glyphs listing holds its counterpart.
  That listing is already taken whenever such a target is not_found, so no extra resolve runs.
  A member declared in any other unit still blocks, with the same detail as before.
- `planglyph.BindHandles`: a handle whose expected glyph the delta does not hold binds to its counterpart when the delta holds that.
  The plan is rewritten to the glyph the card created, so drift detection and later batches see the real declaration instead of a `bind-count-mismatch`.

## Design decisions

- Both directions are handled, since a plan may equally name `<dir>_test` for a test written in-package.
- The fix is at record time rather than a plan validator rewrite, because a new external test file does not exist yet when the plan is validated.
- The PATTERN-glyph-conversion-chokepoint background now names `GoPackageCounterpart` beside `goPackageDir` as the only sanctioned Go unit adjustments.
  The PATTERN.md entry is unchanged.

## Tests

- `TestGoPackageCounterpart` covers both directions, method members, and the cases with no counterpart.
- `TestDoneChecks_Verdicts` gains rows for a package test landed in the external test package and the reverse; the existing "declared in another unit" row still blocks.
- `TestBindHandles_HandleBindsToTheOtherPackageOfItsDirectory` covers binding and the rewrite in both directions.
