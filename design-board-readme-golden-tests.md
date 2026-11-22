# The board README renderer carries tests for pure cosmetics

`board-readme-golden-tests` · Note · bug

The operator's call: a file that only renders the board's cosmetic markdown needs no tests.
Tests there are clutter.

Seen in `readme-declutter`: the format change touched about 200 lines of `internal/boardengine/render.go`,
and about 480 lines of `internal/boardengine/render_test.go`.
The goldens in `TestRenderReadmeGolden` spell out every rendered README byte by byte as Go strings,
so every format change is a hand rewrite of the tests.
`view_test.go` also parses the README's headings, so it breaks on a format change too.

Left for whoever takes this: some tests in the same files pin behaviour that is not cosmetic,
such as `RenderToDisk` rewriting a file in place and its manifest cleanup,
and `board-data.js` placing each entry where the README shows it.
