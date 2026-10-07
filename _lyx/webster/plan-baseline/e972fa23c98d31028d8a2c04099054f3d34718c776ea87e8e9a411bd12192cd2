# Card 3 — websterengine-index-seam

**Edit:**
- `internal/websterengine/geometry.go`
- `internal/websterengine/git.go`
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/gitfake_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/wiring.go`
- `internal/loomcli/wiring.go`

**Uses:**
- `plan:internal/planindex#Index`
- `plan:internal/planglyph#NewIndex`

**Intent:**
Decision D8's seam for `websterengine`: its production code stops importing `internal/planglyph` and `quarry/quarry`, so `configreg`, `hubgeom`, `standalonegeom`, `cliwire` and every CLI test binary that reaches `websterengine` through them stop linking tree-sitter.

`Geometry` gains `Index planindex.Index`.
Every `planglyph` call in `beginbatch.go`, `recordbatch.go` and `runlevel.go` goes through `Geom.Index`, and `Git.Delta` leaves the `Git` interface: `postBatchChecks` takes its one batch delta from `Index.Delta` and calls `BindHandles`, `ScopeGuard` and `DetectDrift` on it, keeping today's order and both fingerprint restamps.
Finding types and `ErrQuarryUnavailable` are named through `planindex`.
A nil `Index` reached at a call is a returned error naming the missing wiring, never a panic.

The CLI layer wires the real index: `webstercli/wiring.go` sets `Index: planglyph.NewIndex()` on both the hub and the standalone geometry, and `loomcli/wiring.go` sets it on the geometry it builds for the Webster row.
`hubgeom` and `standalonegeom` leave the field empty, since they sit below `cliwire` and must not link the index.

Tests: `gitfake_test.go`'s `fakeGit` drops `Delta`, and the delta it carried moves to a fake `planindex.Delta`.
A test that exercises the real resolve pass sets `Geom.Index` to `planglyph.NewIndex()`, so `websterengine`'s own test binary still links tree-sitter and stays on card 5's allowed list;
its production package does not.
No test expectation changes: this is a seam refactor, and the existing tests are its coverage.

**ImpactSummary:** Every `websterengine` caller outside the CLI layer is unaffected; `webstercli` and `loomcli` gain one wiring line each, and `websterengine` production drops its last tree-sitter import.
