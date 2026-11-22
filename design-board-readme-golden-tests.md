# The board README renderer carries tests for pure cosmetics

`board-readme-golden-tests` · Task · bug, board

## Decision (operator, 2026-11-22)

Output only a human reads, such as the rendered README, gets no test of its layout or wording.
The general rule goes into `scribe:testing` through the scribe hub; this entry runs as darn light once that rule has landed.

## What to do

- Delete the README layout goldens in `internal/boardengine/render_test.go` and the heading parsing in `view_test.go`; do not move them to testdata files.
- Keep the tests of behaviour other code relies on, without pinning the layout: `RenderToDisk` rewriting a file in place and its manifest cleanup, and `board-data.js` holding each entry, which `board.html` reads.
- Sweep the repository for other renderers of human-read output carrying layout goldens, and delete those too.

## What was seen

In `readme-declutter` the format change touched about 200 lines of `render.go` and about 480 lines of `render_test.go`.
On main, `render_test.go` is longer than `render.go`: each run covered its format change with a new golden, and reviewers flag a change without a test.
