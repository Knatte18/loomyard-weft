# Card 16 — Dobbies start nothing while a move record stands

**Edit:**
- `internal/dobbyengine#Geometry`
- `internal/dobbyengine#StartDeps`
- `internal/dobbyengine#ensureOne`
- `internal/dobbyengine#daemon.tick`
- `internal/dobbyengine/doc.go`
- `internal/hubgeom#DobbyGeometry`
- `internal/hubgeom#DobbyStart`
- `internal/dobbyengine#fakeStart`
- `internal/dobbyengine#fakeStart.deps`
- `internal/dobbyengine#TestEnsureLive`
- `internal/hubgeom#TestDobbyGeometry`

**Create:**
- `plan:internal/dobbyengine#TestRun_NoPassWhileAMoveStands` -> `func TestRun_NoPassWhileAMoveStands(t *testing.T)`

**Uses:**
- `plan:internal/fabricengine#MoveRecordPath`
- `plan:internal/fabricengine#MoveInProgress`
- `plan:internal/lyxcwd#NamedWorktree`
- `internal/dobbyengine#EnsureLive`
- `internal/dobbyengine#daemon.drivePasses`
- `internal/dobbyengine#daemon.startReady`
- `internal/dobbyengine#daemon.tidyStep`
- `internal/dobbyengine#newLoopHarness`

**Intent:**
D7 step 1's dobby gate, through seams `hubgeom` fills, so `dobbyengine` stays told its paths (`PATTERN-cwd-resolution`'s told geometry).
`Geometry` gains `MoveRecords []string`: the move-record paths whose standing stops this dobby, its own worktree's weft record and, for every dobby, the prime weft's record, filled by `DobbyGeometry` through `fabricengine.MoveRecordPath` of each weft (`NamedWorktree` for the beam's own).
`daemon.tick` checks them with `os.Stat` before `drivePasses`; while any exists it skips `drivePasses`, `startReady` and `tidyStep` that tick, since the board-tidy agent is a strand whose state sits at the anchor, logs that once per episode, and goes on to its record keeping and landing count.
An empty `MoveRecords`, as a hand-built `Geometry` leaves it, gates nothing.
`StartDeps` gains `MoveInProgress func(worktree string) (wayForward string, standing bool, err error)`, which `DobbyStart` fills from `fabricengine.MoveInProgress` over the worktree's own weft and the prime's.
`ensureOne` asks it before `Live`: a standing record reports the worktree `OutcomeRefused` with the way forward as `Detail` and starts nothing, so `lyx beam start` and `lyx orch start` start no dobby while a change stands; a read error reports `OutcomeFailed`.
A nil `MoveInProgress` reads as no record standing.
`dobbyengine/doc.go` says both.

Tests:
`TestEnsureLive` gains rows on `fakeStart` (a `moving` map): a moving beam is refused with the way forward and not spawned while the others start; a moving worktree's read error reports it failed.
`TestRun_NoPassWhileAMoveStands` (tier0, on `newLoopHarness` with its `Runs` and `SpawnPass` deps faked to hold one due run, and a tidy fake counting its spawns) drives ticks with a record file present, then removed: no pass, task or tidy agent starts while it stands, and the next tick after its removal starts the due pass.
`TestDobbyGeometry` asserts `MoveRecords` for the prime and for a beam, and drives `DobbyStart(prime).MoveInProgress` over a temp hub: a record written at the prime weft's `MoveRecordPath` reads standing for the prime and for a beam, and one at the beam's weft reads standing for that beam only.

**ImpactSummary:** two struct fields in `dobbyengine` and their `hubgeom` fillers, one guard in the tick and one in the start; with no record on disk dobbies behave as today, and the in-package fake gains the seam.
