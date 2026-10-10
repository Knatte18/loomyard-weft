# Card 5 — hubgeom-gate-slots

**Create:**
- `plan:internal/hubgeom#GateSlots` -> `func GateSlots(l *lyxcwd.Location) *gateslot.Pool`

**Edit:**
- `internal/websterengine#Geometry`
- `internal/hubgeom#WebsterGeometry`
- `internal/hubgeom/webstergeom_test.go`

**Uses:**
- `plan:internal/gateslot#Dir`
- `plan:internal/gateslot#LoadConfig`
- `plan:internal/gateslot#Config.Limits`
- `internal/fabricengine#BoardDir`

**Intent:**
Give the hub-mode teller the one constructor of a hub's slot pool, so every Go-side site is told the same pool and none resolves the board dir itself (`PATTERN-told-geometry`).
`GateSlots(l)` returns a pool over `gateslot.Dir(fabricengine.BoardDir(l.HubPath))` whose `Limits` closure loads `gate.yaml` strictly from the board dir at each acquire and converts it with `Config.Limits`; a load failure surfaces at acquire as the error, naming the file.
`websterengine.Geometry` gains a `GateSlots *gateslot.Pool` field, documented as the told pool every webster gate run acquires from, nil in standalone mode where there is no hub.
`WebsterGeometry` fills it with `GateSlots(l)`; `standalonegeom` leaves it nil.
`webstergeom_test.go` gains assertions that the field's directory is the board dir's slot directory and that its limits read the board dir's `gate.yaml`.

**ImpactSummary:** Adds one field to webster's geometry, filled in hub mode only; nothing reads it until the next card.
