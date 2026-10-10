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

## Quarry's Enclose as delivered (quarry PR #36, 2026-10-08)

- Released in quarry v0.3.0: `(*Repo).Enclose(locs)` and `(*Repo).EncloseAt(rev, locs)`, with `EncloseResult`, `EncloseReasons` and `RenderEncloseJSON`; paths are relative to the repo root, not the cwd, and an unknown rev fails the whole call.
- They take location spellings (`path:N`, `path:N-M`, `path:N:col`), not structs; Loomyard formats tool-output locations into them.
- Rejections carry a closed reason set (`EncloseReasons`), including `unaddressable` for a `.go` file in the repo root or a path containing `#`; Loomyard cites `path:line` for any rejected item.
- A `found` glyph may not Resolve afterwards (an explicitly named gitignored file, a file without a package clause, a path through a symlinked directory; `quarry/enclose.go`). Loomyard treats a Resolve miss after Enclose as "no current span", never as an error, the same as a member the fixer deleted or renamed.
- Loomyard pins quarry v0.2.1; this task bumps `go.mod` to v0.3.0, which has no other breaking change.

## Review kick-start (2026-10-10)

The fan-lab research (`fan-master-preread`) found solo `opus[medium]` the only medium-effort reviewer to find the BLOCKING finding, at 225 s per Discussion round; a pack of the code the target cites (22 of 25 files the forks needed) is the candidate to cut that time.

One fan-lab run tested it by hand: the bodies of the 35 symbols the target cites, resolved with `lyx quarry resolve` (about 20k tokens), in a file the solo reviewer reads first.
It was slower and dearer than plain solo (312 s against 225 s, $1.93 against $1.45 review side) and missed the BLOCKING finding; the reviewer read the pack and then made about 20 tool calls of its own, so the pack added cost without replacing exploration.
On that single run the pack does not pay off for review; its use for verify failures and finding locations above stands on its own.
