# The board README renderer carries tests for pure cosmetics

`board-readme-golden-tests` · Note · bug, board

## Decision (operator, 2026-11-22)

A function that only renders the board's markdown for reading carries no tests: a wrong rendering breaks no production code, and a test that pins its layout only makes every format change a rewrite.
Delete the README layout goldens; do not move them to testdata files.

## What was seen

In `readme-declutter` the format change touched about 200 lines of `internal/boardengine/render.go` and about 480 lines of `internal/boardengine/render_test.go`.
The goldens in `TestRenderReadmeGolden` spell out every rendered README byte as Go strings.
`view_test.go` also parses the README's headings, so it breaks on a format change too.

## What stays

Tests of behaviour other code relies on stay, without pinning the layout:
`RenderToDisk` rewriting a file in place and its manifest cleanup, and `board-data.js` holding each entry, which `board.html` reads.
