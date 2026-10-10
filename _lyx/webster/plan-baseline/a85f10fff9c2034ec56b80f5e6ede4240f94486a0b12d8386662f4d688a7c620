# Card 5 — hubgeom-gate-slots

**Create:**
- `internal/hubgeom#GateSlots`

**Edit:**
- `internal/websterengine#Geometry`
- `internal/hubgeom#WebsterGeometry`
- `internal/hubgeom/webstergeom_test.go`
- `internal/hubgeom/doc.go`
- `internal/websterengine/doc.go`

**Uses:**
- `internal/gateslot#Dir`
- `internal/gateslot#WaitDir`
- `internal/gateslot#LoadConfig`
- `internal/gateslot#Config.Limits`
- `internal/fabricengine#BoardDir`

**Intent:**
Give the hub-mode teller the one constructor of a hub's slot pool, so every Go-side site is told the same pool and none resolves the board dir itself (`PATTERN-told-geometry`).
`GateSlots(l)` returns a pool over `gateslot.Dir(fabricengine.BoardDir(l.HubPath))` whose `Limits` closure loads `gate.yaml` strictly from the board dir at each acquire and converts it with `Config.Limits`; a load failure surfaces at acquire as the error, naming the file.
`websterengine.Geometry` gains a `GateSlots *gateslot.Pool` field, documented as the told pool every webster gate run acquires from, nil in standalone mode where there is no hub.
It also gains a `GateWaitDir string` field, documented as the told per-worktree directory a slotted card-verify rerun writes its wait record into, empty in standalone mode, where no pool means no wait, so webster's engine never joins that directory itself (`PATTERN-told-geometry`).
`WebsterGeometry` fills them with `GateSlots(l)` and `gateslot.WaitDir(anchorPath)`; `standalonegeom` leaves both zero.
`webstergeom_test.go` gains assertions that the pool's directory is the board dir's slot directory, that its limits read the board dir's `gate.yaml`, and that `GateWaitDir` is the anchor's wait directory.
`internal/hubgeom/doc.go`'s contract paragraph, which enumerates the package's tellers, gains `GateSlots`: the one constructor of a hub's gate-slot pool, over the board dir's slot directory, with its limits read from the board dir's `gate.yaml` at each acquire.
`internal/websterengine/doc.go`'s sentence that points at geometry.go for "what each of Geometry's eight fields means", already a stale tally and falser with two more fields, drops the count and points at geometry.go for what each of Geometry's fields means.

**ImpactSummary:** Adds two fields to webster's geometry, filled in hub mode only; nothing reads them until the next card.
