# Card 7 — webster Geometry carries the verify directory

**Edit:**
- `internal/websterengine/geometry.go`
- `internal/hubgeom/webstergeom.go`
- `internal/standalonegeom/webstergeom.go`
- `internal/hubgeom/webstergeom_test.go`
- `internal/standalonegeom/standalonegeom_test.go`
- `internal/websterengine/webstergeom_test.go`

**Uses:**
- `internal/verifytree/verify.go`

**Intent:**
`websterengine.Geometry` gains `VerifyDir`, the told directory of the worktree's verified-tree record, running marker and verify log.
`hubgeom.WebsterGeometry` sets it to `verifytree.Dir(anchorPath)`, the same directory loom's landing rows use (card 6), so a tree the webster gate verified is skipped by Publish;
`standalonegeom.WebsterGeometry` sets it to `verifytree.Dir(stateDir)`.
Per the Told-Geometry Invariant, websterengine never derives it; the webster gate (card 15) and `lyx webster verify` (card 18) read it.
The geometry doc comment's field count and the three geometry tests follow the new field.

**ImpactSummary:** one added Geometry field filled by both tellers; no consumer reads it yet.
