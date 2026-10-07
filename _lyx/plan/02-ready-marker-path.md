# Card 2 — One function deriving the review-ready marker path

**Create:**
- `plan:internal/burlermarker#Path` -> `func Path(root, base, reviewPath string) (string, error)`
- `internal/burlermarker/doc.go`
- `internal/burlermarker/marker.go`
- `internal/burlermarker/marker_test.go`

**Intent:**
Add package `internal/burlermarker` with `Path`, the single function D5 requires: it derives the machine-local ready marker of a burler round from the round's review path, and both `shedadapters.BurlerProducer` and `lyx burler run` call it (card 6), so any process re-derives the same marker and no caller builds its own.
It lives outside `internal/burlerengine` per `PATTERN-told-geometry`: the engine is told the result with the round's other paths.

Rule: `reviewPath` is resolved against `root` when relative;
it must lie inside `root`, else an error names both paths;
its path relative to `root` drops a leading `lyxdirs.LyxDirName` segment when it has one;
the marker is that relative path joined under `filepath.Join(base, lyxdirs.DotLyxDirName)`, with the suffix `.ready` on the file name.
In hub geometry `root` and `base` are both the anchor, so `<anchor>/_lyx/reviews/webster/round-2-review.md` maps to `<anchor>/.lyx/reviews/webster/round-2-review.md.ready`, the mirrored ephemeral subpath (`PATTERN-durable-vs-ephemeral-state`).
In standalone geometry `root` is the reviewed target and `base` the told state directory, so the marker sits under `.lyx` in the state directory at the review file's path relative to the target.
The package imports only the standard library and `internal/lyxdirs`, and spells neither literal (`PATTERN-lyxdirs-single-declarer`).
`doc.go` carries the package doc stating the rule and its two callers.

`marker_test.go` is one table-driven test: a hub path under `_lyx`, a hub-root path not under `_lyx`, a relative review path, a standalone target/state-dir pair, and a path outside `root` (error).
