# Card 2 — One function deriving the review-ready marker path

**Create:**
- `internal/burlermarker#Path`
- `internal/burlermarker/doc.go`
- `internal/burlermarker/marker.go`
- `internal/burlermarker/marker_test.go`

**Edit:**
- `docs/overview.md`
- `cmd/lyx/pathrules_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add package `internal/burlermarker` with `Path`, the single function D5 requires: it derives the machine-local ready marker of a burler round from the round's review path, and both `shedadapters.BurlerProducer` and `lyx burler run` call it (card 7), so any process re-derives the same marker and no caller builds its own.
It lives outside `internal/burlerengine` per `PATTERN-told-geometry`: the engine is told the result with the round's other paths.

Rule: `reviewPath` is resolved against `root` when relative;
it must lie inside `root`, else an error names both paths and ends with its way forward, to place the review path under `root`;
its path relative to `root` drops a leading `lyxdirs.LyxDirName` segment when it has one;
the marker is that relative path joined under `filepath.Join(base, lyxdirs.DotLyxDirName)`, with the suffix `.ready` on the file name.
In hub geometry `root` and `base` are both the anchor, so `<anchor>/_lyx/reviews/webster/round-2-review.md` maps to `<anchor>/.lyx/reviews/webster/round-2-review.md.ready`, the mirrored ephemeral subpath (`PATTERN-durable-vs-ephemeral-state`).
In standalone geometry `root` is the reviewed target and `base` the told state directory, so the marker sits under `.lyx` in the state directory at the review file's path relative to the target.
The package imports only the standard library and `internal/lyxdirs`, and spells neither literal (`PATTERN-lyxdirs-single-declarer`).
`doc.go` carries the package doc stating the rule and its two callers.
`docs/overview.md`'s module tree gains the `internal/burlermarker` line.

This is a refusal a standalone `lyx burler run` can now reach, since a profile's `review-path` may be absolute and outside the target today (card 7 states it in the verb's help).

`marker_test.go` is one table-driven test: a hub path under `_lyx`, a hub-root path not under `_lyx`, a relative review path, a standalone target/state-dir pair, and a path outside `root` (an error naming both paths and the way forward).
The package is new, so `cmd/lyx/testdata/test-budget.yaml` gains an `internal/burlermarker` row of one (`TestTestBudget_PackagesStayWithinBudget` fails on a package with tests and no row).

`Path` is a new never-tracked path constructor, so `cmd/lyx/pathrules_test.go`'s `pathRules` table gains a `classTransient` row for it, over a review path given relative to the anchor (`filepath.Join(lyxdirs.LyxDirName, "reviews", "x-review.md")`, never built from `l.AnchorPath()`, which is itself relative in this table's fixtures and would be resolved under `root` a second time) with `l.AnchorPath()` as both `root` and `base`, so the existing subpath-anchored fixture proves the marker stays under the anchor-based `.lyx` root and the leading `_lyx` segment is dropped.
The row sets no `mirrors`, since the guard compares byte-for-byte after the `_lyx` to `.lyx` rewrite and the marker adds the `.ready` suffix.
It is a row of the existing `TestPathRules`, so the budget is unchanged.

**ImpactSummary:** one added line in the `docs/overview.md` module tree, one new row in the path-rules table and one new budget row; no other file reads any of them.
