# Card 6 — burler-geometry-names

**Edit:**
- `internal/burlerengine#Geometry`
- `internal/hubgeom#BurlerGeometry` -> `func BurlerGeometry(l *lyxcwd.Location) (burlerengine.Geometry, error)`
- `internal/standalonegeom#BurlerGeometry` -> `func BurlerGeometry(target, stateDir, hash8 string) burlerengine.Geometry`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/burlercli#burlerCLI.wireStandalone`
- `internal/loomcli#loomCLI.wire`
- `internal/burlerengine/smoke_round_test.go`
- `internal/burlerengine/smoke_cluster_test.go`
- `internal/hubgeom/hubgeom_test.go`
- `internal/standalonegeom/standalonegeom_test.go`
- `internal/hubgeom/doc.go`

**Create:**
- `plan:internal/hubgeom#burlerGeometry` -> `func burlerGeometry(l *lyxcwd.Location, prime bool) burlerengine.Geometry`

**Uses:**
- `internal/hubgeom#isPrimeWorktree`
- `internal/hubgeom#reedGeometry`

**Intent:**
Tell burler the hub shortname and task slug its seats' strand names are formed from (decision D2), ahead of the seat table that reads them (card 8).

`burlerengine.Geometry` gains `Shortname` and `Slug`, documented like `seatengine.Geometry`'s fields of the same names: the shortname and slug the round's agent names are formed from, an empty slug giving the two-segment form.
burlerengine derives neither (`PATTERN-told-geometry`).

`hubgeom.BurlerGeometry` now tells the prime from a task worktree through `isPrimeWorktree`, as `ReedGeometry` does, and returns its error the same way, since telling a task worktree the prime's names would misname its strands.
The rest moves into `burlerGeometry(l, prime)`, which fills the existing fields as today plus `Shortname` and `Slug` exactly as `reedGeometry` fills `NameShortname` and `NameSlug` (the recorded hub shortname; the worktree name for a task worktree, empty for the prime), so the names the seat engine forms match the names reed gives the strands.
`standalonegeom.BurlerGeometry` takes the session's `hash8` and fills `Shortname` with `agentname.StandaloneShortname(hash8)` and `Slug` empty, matching `standalonegeom.ReedGeometry`; its doc comment drops the "takes no hash8" sentence.

Callers move with the new signatures: `wireHub` and `loomCLI.wire` return the hub builder's error as their surrounding wiring errors return, `wireStandalone` passes `res.Hash8`, and both `llm`-tier burler smoke tests take the error with `t.Fatalf`.

Tests:
- `hubgeom_test.go`: `TestBurlerGeometry` and `TestBurlerGeometry_ParentNameFromOriginRecord` call `burlerGeometry` with prime false, since their fixtures carry no `.git` entry, and `TestBurlerGeometry` asserts `Shortname` is the recorded shortname and `Slug` the worktree name, and that prime true leaves `Slug` empty.
- `standalonegeom_test.go`: `TestBurlerGeometry` asserts `Shortname` is `agentname.StandaloneShortname(hash8)` and `Slug` is empty.

Note in `internal/hubgeom/doc.go` that `BurlerGeometry` tells the shortname and slug as `ReedGeometry` does.

**ImpactSummary:** Two geometry builders re-signed with five call sites, all wiring or tests; burlerengine reads the new fields only from card 8.
