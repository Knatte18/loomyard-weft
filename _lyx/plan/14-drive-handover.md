# Card 14 — drive-handover

**Edit:**
- `plan:internal/loomcli#goDriverDeps`
- `plan:internal/loomcli#goDriver.run`
- `plan:internal/loomcli#goDriver.facts`
- `plan:internal/loomcli#TestGoDriver_Settle`

**Create:**
- `plan:internal/loomcli#goDriver.handOver` -> `func (d *goDriver) handOver(ctx context.Context, stop driveStop) error`

**Uses:**
- `plan:internal/loomcli#binaryEligible`
- `internal/loomengine#SettleHandOver`

**Intent:**
D5: at a binary-eligible stop whose last step ran a build other than the `lyx` now on disk, the Go driver hands the run to a fresh driver of the on-disk build instead of settling it with old code.
The code lands in `internal/loomcli/drive.go`.

`goDriverDeps` gains five seams:
- `OnDiskBuild() (buildvcs.Identity, error)`: `buildvcs.OfFile` over the `lyx` executable path resolved once at drive start;
- `BuildChanged(onDisk buildvcs.Identity) bool`: `shedverbs.LastStepBuildChanged` over the run's steps directory;
- `DirtyPaths() ([]string, error)`: the task pair's uncommitted or untracked paths outside the run's own records directories;
- `ReleaseLock() error`: releases the driver lock;
- `SpawnSuccessor() error`: starts a detached `lyx loom drive`.
`facts` reads `OnDiskBuild` and `BuildChanged` only for a `binaryEligible` stop, and sets `HandedOver` when the settle records hold a `hand-over` record for the stop key whose `Build` is the on-disk build's label.
An unreadable on-disk build reads as unchanged, with one Warn.
So there is one hand-over per stop key and build, and any difference, a rollback included, counts.

`handOver`, in this order:
1. Holds any pause (D11).
2. Reads `DirtyPaths`. A non-empty list hands back with the paths in the stop report and spawns nothing.
3. Writes the `hand-over` settle record (settler `go-rule`, `Build` the on-disk label).
4. Runs `handBackRun` with a report that names the hand-over, so the stop report and the park marker are on disk and an unwatched run's parent receives the D8 line naming the hand-over before the release.
5. Calls `ReleaseLock`, then `SpawnSuccessor`, logging the spawn, and exits.
The fresh driver removes the park marker on start (card 15) and re-steps.
A failed spawn, or a crash after the release, leaves a parked run with a stop report.
When batten watches, card 20's re-step picks the run up; when nothing watches, the parent already has the line.
Bound: the hand-over runs only at a binary-eligible stop, once per stop key and build, never over a dirty tree, and never under a pause.

Tests are rows of `TestGoDriver_Settle`, with fake build, dirty-path, release and spawn columns that record their call order:
- a changed build with a clean tree writes the `hand-over` record, the report and the park marker, calls `ReleaseLock` before `SpawnSuccessor`, and exits;
- the same stop key and build a second time does not hand over again and goes on to the tackler;
- a dirty tree hands back naming its paths and spawns nothing;
- a failing `SpawnSuccessor` leaves the report and the park marker on disk;
- an unwatched run reaches `Notify` before `ReleaseLock`;
- an unchanged build, or a stop that is not binary-eligible such as `awaiting`, takes its usual row;
- a pause covering the slug holds the hand-over until `PauseHold` returns.

**ImpactSummary:** extends the unexported Go driver loop and its test table with the hand-over; nothing outside the driver changes.
