# Kick-start packs from tool locations through quarry Enclose

`enclose-kickstart` · Note · enhancement, quarry

Operator, 2026-10-08: quarry gains `Enclose`, written as quarry's task `enclose-locations` (in the quarry repo; the operator spawns it).
It maps `file:line` ranges from tool output to the innermost enclosing member glyphs, with the same Symbol data `Resolve` returns, also at a git revision (`quarry enclose [--rev R]`).
Markdown is out of its scope: a `.md` location is a per-item rejection.

Use in Loomyard once it lands, building on the kick-start pack:
this waits for task `plan-glyph-tuning`, which builds the Webster pack behind a config switch, and for its with/without measurement to show the pack helps (operator, 2026-10-11).
Operator decision 2026-10-15: this waits on the `tools/forkaudit` measurement of the per-card pack (landed with `kickstart-per-card`), taken after 4-5 Webster runs on it and again after `webster-master-no-code` lands, since that changes what forks inherit.

- Verify failures (`go test` failure lines, `go vet`, compile errors) give the recovery agent or fixer a pack of the failing members.
- Comment-lint findings and Webster audit findings carry glyphs instead of lines.
- A review finding that cites `path:line` despite the glyph rule is mapped at the reviewed revision, then resolved in the working tree, so the fixer's own edits never shift it.

Glyph↔path conversion stays in quarry; Loomyard never intersects lines with symbol spans itself.

## Quarry's Enclose as delivered (quarry PR #36, 2026-10-08)

- Released in quarry v0.3.0: `(*Repo).Enclose(locs)` and `(*Repo).EncloseAt(rev, locs)`, with `EncloseResult`, `EncloseReasons` and `RenderEncloseJSON`; paths are relative to the repo root, not the cwd, and an unknown rev fails the whole call.
- They take location spellings (`path:N`, `path:N-M`, `path:N:col`), not structs; Loomyard formats tool-output locations into them.
- Rejections carry a closed reason set (`EncloseReasons`), including `unaddressable` for a `.go` file in the repo root or a path containing `#`; Loomyard cites `path:line` for any rejected item.
- A `found` glyph may not Resolve afterwards (an explicitly named gitignored file, a file without a package clause, a path through a symlinked directory; `quarry/enclose.go`). Loomyard treats a Resolve miss after Enclose as "no current span", never as an error, the same as a member the fixer deleted or renamed.
- Loomyard pins quarry v0.2.1; this task bumps `go.mod` to v0.3.0, which has no other breaking change.
