# Kick-start packs from tool locations through quarry Enclose

`enclose-kickstart` · Note · enhancement, quarry

Operator, 2026-10-08: quarry gains `Enclose`, written as quarry's task `enclose-locations` (in the quarry repo; the operator spawns it).
It maps `file:line` ranges from tool output to the innermost enclosing member glyphs, with the same Symbol data `Resolve` returns, also at a git revision (`quarry enclose [--rev R]`).
Markdown is out of its scope: a `.md` location is a per-item rejection.

Use in Loomyard once it lands, building on the kick-start pack:

- Verify failures (`go test` failure lines, `go vet`, compile errors) give the recovery agent or fixer a pack of the failing members.
- Comment-lint findings and Webster audit findings carry glyphs instead of lines.
- A review finding that cites `path:line` despite the glyph rule is mapped at the reviewed revision, then resolved in the working tree, so the fixer's own edits never shift it.

Glyph↔path conversion stays in quarry; Loomyard never intersects lines with symbol spans itself.
