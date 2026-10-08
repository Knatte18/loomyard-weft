# Card 8 — Hub geometry for the config walk

**Create:**
- `plan:internal/hubgeom#ReconcileGeometry` -> `func ReconcileGeometry(l *lyxcwd.Location) (hubreconcile.Geometry, bool)`

**Edit:**
- `internal/hubgeom/hubgeom_test.go`
- `internal/hubgeom/doc.go`

**Uses:**
- `internal/preflight#BoardLyxPresent`
- `plan:internal/hubreconcile#Geometry`

**Intent:**
`internal/hubgeom` is the hub-mode constructor of every engine's `Geometry` struct (`PATTERN-told-geometry`), so it builds the one the start verbs (card 9, card 10) and the `Preflight` row (card 11) hand to `hubreconcile.Ensure`; the four call sites then share one construction instead of each joining paths.

`ReconcileGeometry` returns `hubreconcile.Geometry{BoardDir: fabricengine.BoardDir(l.HubPath), WorktreePath: l.WorktreePath()}` and true when `preflight.BoardLyxPresent(l)` reports the hub's board-level `_lyx` dir, and the zero `Geometry` and false otherwise, so a standalone repository (Decision 3) never reconciles.
`doc.go`'s constructor list names it.

Tests: `hubgeom_test.go` gains a table test over a `t.TempDir()` hub layout with a hand-built `*lyxcwd.Location`: a hub with `_board/_lyx` present returns true with exactly those two paths; the same layout without `_board/_lyx` returns false and the zero value.

**ImpactSummary:** One additive constructor in hubgeom, which now imports hubreconcile; no existing constructor changes.

**Commit:** 8: hubgeom: build the hub config walk's geometry
