# Card 8 — declare the driver park marker in shedrun

**Create:**
- `plan:internal/shedrun#ParkMarkerFileName` -> `const ParkMarkerFileName = "driver-parked"`
- `plan:internal/shedrun#ParkMarker` -> `func ParkMarker(l *lyxcwd.Location, runID string) string`

**Edit:**
- `internal/shedrun/paths.go`
- `internal/shedrun/paths_test.go`

**Uses:**
- `internal/shedrun#ScratchDir`

**Intent:**
Declare the park marker's filename and path once, in `internal/shedrun` (decision record D7), per the Shed Run-Directory and Lyxdirs Single-Declarer Invariants: `loomcli` composes the path through this constructor and never spells a segment itself.

`ParkMarkerFileName` is exported because the ly-drive skill names the same filename and a test pins the two together (card 11).
Its doc comment says so, names the file's content (the path of the stop report the driver parked on), and names its lifecycle: a loom-launched ly-drive driver writes it at a hand-back, the driver removes it before a self-initiated re-step, and `lyx loom start` removes it when it resumes the driver or before it spawns a fresh one.

`ParkMarker` returns `ScratchDir(l, runID)` joined with `ParkMarkerFileName`, so the marker sits in the directory the step envelope reports as `scratch_dir`, under `.lyx` per the Durable-vs-Ephemeral State Invariant.
The file header's list of what `paths.go` declares mentions it among the ephemeral paths.

Test (`paths_test.go`): `ParkMarker` equals `filepath.Join(ScratchDir(l, runID), ParkMarkerFileName)` for a slug run-id and for the `self` alias, and sits under the location's `.lyx` tree, beside the existing constructors' own cases.

**ImpactSummary:** Adds one exported constant and one path constructor to `shedrun`; no existing path moves.

**Commit:** 8: shedrun: declare the ly-drive park marker path
