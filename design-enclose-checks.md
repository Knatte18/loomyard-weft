# Use quarry Enclose for mechanical checks: verify failures to plan cards

`enclose-checks` · Note · enhancement, quarry, webster, loom

Operator, 2026-10-08: quarry gains `Enclose`, written as quarry's task `enclose-locations` (in the quarry repo; the operator spawns it).
It maps `file:line` ranges from tool output to the innermost enclosing member glyphs, with the same Symbol data `Resolve` returns, also at a git revision (`quarry enclose [--rev R]`).
Markdown is out of its scope: a `.md` location is a per-item rejection.

## Change of direction, 2026-10-17

The original use was a kick-start pack of the failing members for recovery agents and fixers.
The pack measurement this waited on (operator decisions 2026-10-11 and 2026-10-15) came in with `docs/research/glyph-audit-2026-10-17/`: forks read code themselves whatever the pack holds, and note `kickstart-pack-remove` records the operator decision to remove the pack.
A pack of failing members is dropped with it: a fork given `file:line` reads around it in one call.

Operator view, not a decision: Enclose can be useful for mechanical checks, where Loomyard's own code needs to know which member a line belongs to, with no agent in the loop.

## Orch recommendation, not a decision

First use, which changes no flow: a verify failure at Merge-In or Publish is mapped through Enclose to its member glyph, then to the plan card whose targets hold that glyph and the batch and commit that ran the card (`_lyx/webster/state.json`).
The fixer's brief names that card, its intent and its commit, as a starting point instead of the whole diff.
Later possibilities, each its own decision:

- Re-run only the batch that made the failing change (`lyx webster reset --batch` and a fresh fork with the failure attached), instead of a whole-diff review round.
- A finding that cites `path:line` is mapped to a glyph at the reviewed revision, so later edits never shift it, and it can be checked against the plan's targets.
- A failure in a member no card targets is reported as a scope gap: the plan missed it.
- Test, vet and lint failures are grouped per member, so duplicate findings fold.

Measure that a fixer uses the named card before building more, as the pack showed agents may read and ignore such hints.

Glyph↔path conversion stays in quarry; Loomyard never intersects lines with symbol spans itself.

## Quarry's Enclose as delivered (quarry PR #36, 2026-10-08)

- Released in quarry v0.3.0: `(*Repo).Enclose(locs)` and `(*Repo).EncloseAt(rev, locs)`, with `EncloseResult`, `EncloseReasons` and `RenderEncloseJSON`; paths are relative to the repo root, not the cwd, and an unknown rev fails the whole call.
- They take location spellings (`path:N`, `path:N-M`, `path:N:col`), not structs; Loomyard formats tool-output locations into them.
- Rejections carry a closed reason set (`EncloseReasons`), including `unaddressable` for a `.go` file in the repo root or a path containing `#`; Loomyard cites `path:line` for any rejected item.
- A `found` glyph may not Resolve afterwards (an explicitly named gitignored file, a file without a package clause, a path through a symlinked directory; `quarry/enclose.go`). Loomyard treats a Resolve miss after Enclose as "no current span", never as an error, the same as a member the fixer deleted or renamed.
- Loomyard pins quarry v0.2.1; this task bumps `go.mod` to v0.3.0, which has no other breaking change.
