# Card 15 — pawl-engine

**Create:**
- `internal/pawlengine/doc.go`
- `plan:internal/pawlengine#NameBeamUp` -> `const NameBeamUp = "Beam-Up"`
- `plan:internal/pawlengine#NameBeamAwait` -> `const NameBeamAwait = "Beam-Await"`
- `plan:internal/pawlengine#NameBeamDown` -> `const NameBeamDown = "Beam-Down"`
- `plan:internal/pawlengine#LandRequestFileName` -> `const LandRequestFileName = "land-request.json"`
- `plan:internal/pawlengine#DobbyStopMarkerName` -> `const DobbyStopMarkerName = "dobby-stop"`
- `plan:internal/pawlengine#AwaitMarkerName` -> `const AwaitMarkerName = "await-entered"`
- `plan:internal/pawlengine#Paths` -> `type Paths struct`
- `plan:internal/pawlengine#LandRequest` -> `type LandRequest struct`
- `plan:internal/pawlengine#WriteLandRequest` -> `func WriteLandRequest(paths Paths, req LandRequest) error`
- `plan:internal/pawlengine#ReadLandRequest` -> `func ReadLandRequest(paths Paths) (LandRequest, bool, error)`
- `plan:internal/pawlengine#RemoveLandRequest` -> `func RemoveLandRequest(paths Paths) error`
- `plan:internal/pawlengine#Deps` -> `type Deps struct`
- `plan:internal/pawlengine#BeamUp` -> `type BeamUp struct`
- `plan:internal/pawlengine#NewBeamUp` -> `func NewBeamUp(name string, deps Deps) (*BeamUp, error)`
- `plan:internal/pawlengine#BeamUp.Call` -> `func (p *BeamUp) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/pawlengine#BeamAwait` -> `type BeamAwait struct`
- `plan:internal/pawlengine#NewBeamAwait` -> `func NewBeamAwait(name string, deps Deps) (*BeamAwait, error)`
- `plan:internal/pawlengine#BeamAwait.Call` -> `func (p *BeamAwait) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/pawlengine#BeamDown` -> `type BeamDown struct`
- `plan:internal/pawlengine#NewBeamDown` -> `func NewBeamDown(name string, deps Deps) (*BeamDown, error)`
- `plan:internal/pawlengine#BeamDown.Call` -> `func (p *BeamDown) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/pawlengine#nextAwaitWait` -> `func nextAwaitWait(current time.Duration, event bool) time.Duration`
- `plan:internal/pawlengine#TestLandRequest` -> `func TestLandRequest(t *testing.T)`
- `plan:internal/pawlengine#TestBeamUp` -> `func TestBeamUp(t *testing.T)`
- `plan:internal/pawlengine#TestBeamAwait` -> `func TestBeamAwait(t *testing.T)`
- `plan:internal/pawlengine#TestBeamDown` -> `func TestBeamDown(t *testing.T)`
- `plan:internal/pawlengine#TestNextAwaitWait` -> `func TestNextAwaitWait(t *testing.T)`
- `plan:internal/pawlengine#TestMain` -> `func TestMain(m *testing.M)`

**Edit:**
- `cmd/lyx#unboundedWaitFiles`
- `docs/overview.md`

**Intent:**
D8 and D9's three new rows, as a told-geometry engine package: it imports no `lyxcwd` and no CLI package, takes every path and seam from its caller, and records through `internal/state`, so an unknown key survives and a request naming another run is read, never refused (`PATTERN-tolerant-records`).

- `Paths` holds the land request's path, the dobby-stop marker's path and the await marker's path, all under the beam worktree's `.lyx/beam/`, which the hub wiring composes from `lyxdirs` (card 17); the file names are this package's constants.
  `LandRequest` holds `RunID`, the pawl run's seed id, and `RequestedAt`; `WriteLandRequest`, `ReadLandRequest` and `RemoveLandRequest` write, read and remove it, an absent file reading as none and a removal of an absent one succeeding.
- `Deps` holds every seam, each a function the producers check is non-nil when built, a nil one refused naming the field (a fail-loud wiring guard): `Slug`, `Paths`, `RunID`, `SetStatus(producer)` and `EntryProducer()` over the beam project entry's run status, `OwnedElsewhere()`, `MarkBeam`, `SpawnPush`, `RegenerateWorkspace`, `Steps` (the beam steps with no reconcile), `StopDobby`, `StartDobby`, `DobbyLive() (bool, error)` (whether the beam's dobby runs), `Recheck() ([]string, error)` (the re-check's blockers, empty when it passes), `WriteSnapshot`, `Notify(line)` (to the beam's own orch), `Down` (the beam's stop), `Watch` (the request directory's change channel; nil leaves the timer alone), `Now` and `Sleep(ctx, d)`.
- `BeamUp.Call` marks the pair a beam (`MarkBeam`, a no-op when marked), spawns its push, regenerates the hub workspace, removes any land request and the await marker and runs `Steps`, every step idempotent; a failed step is a halting Stuck naming the step's cause and `lyx loom resume`.
- `BeamAwait.Call` holds the run until a land request it accepts, spawning no agent:
  - On entry, with no request standing and `EntryProducer` naming `Parent-Align`, as a crash between the request's removal and Done leaves it, it writes the status naming Beam-Await first, removes a standing marker, and posts one notice that the landing was interrupted, naming `lyx beam start <slug>` and `lyx beam land <slug>`; otherwise it writes the status naming Beam-Await, and a failed write is retried at each wake until one succeeds.
  - It wakes on a `Watch` event and on its timer, which starts at one second and doubles to a minute while idle, returning to a second on an event (`nextAwaitWait`, `PATTERN-daemon-wakeups`), through `Sleep`, so no wake is shorter than a second.
  - Each wake first reads the owner: a beam another machine owns removes any request and returns `shedengine.Yield`, deciding nothing; the batten pass's owner check owner-pauses the run.
  - A request naming another run than `RunID` is removed; one naming this run is accepted: it writes the status naming Parent-Align, so the landing hold stands, and a failed write keeps the request for the next wake; then it writes the marker and calls `StopDobby`, so the dobby starts no new member; then `Recheck`, whose run-at-Beam-Await check is not re-run, since this row is that check.
  - A passing re-check writes the snapshot, the perch's answer key, removes the request, then the marker and the await marker, and is Done.
  - A failing one writes the status naming Beam-Await again, restarts the dobby whenever the marker stands and then removes the marker, removes the request, posts the failure with the blockers and keeps waiting; a failed restart leaves the marker standing, still removes the request, and the posted failure also names `lyx beam start <slug>`, which passes since the status write lifted the hold; the next accepted request restarts the dobby on its own failed re-check, since the marker still stands.
  - A failed write of that Beam-Await status keeps the request and the marker, restarts no dobby and posts nothing, so the hold is never left standing behind a dropped request: the next wake retries the accept, its Parent-Align write, its stop of a dobby already stopped, its re-check and this write, until the write succeeds (D9's writes crossing the hold are retried at each wake).
  - A `Recheck` that returns an error, and a `StopDobby` that fails, are a failed re-check whose one blocker is that error, taking the same path: the hold is lifted by the Beam-Await write, the dobby restarted while the marker stands, the request removed, and the failure posted naming the error and `lyx beam land <slug>` to ask again.
  - A failed `Notify` is logged at Warn with the failure's lines and changes nothing else, since the request is already answered.
    Bound: with no live orch strand recorded for the beam the failure reaches only the driver's log and `lyx batten status <slug>`, which shows the run back at Beam-Await.
  - A crash between the dobby stop and the request's removal re-enters with the request and the marker standing, so the retried accept restarts the dobby on a failed re-check.
  - A run halted or paused after the request's removal is resumed through shed's status write, which names Beam-Await on the entry first, so it re-enters with no request standing and the hold already lifted, but with the dobby the accept stopped still down.
    So the row tells a re-entry from its first entry by the await marker: Beam-Up removes it, Beam-Await writes it on entry when it is absent, and the Done path removes it with the request and the dobby-stop marker.
    On an entry with no request standing that finds the await marker already written, it reads `DobbyLive`, and when the dobby is not live it posts one notice that the beam's dobby is down, naming `lyx beam start <slug>` and `lyx beam land <slug>`.
    The first entry from Beam-Up reads no `DobbyLive`, since the dobby Beam-Up's steps just spawned may not hold its lock yet; neither the entry's status, which shed's status commit has already rewritten to Beam-Await, nor a timer tells the two apart.
    A failed `DobbyLive` read, and a failed write of the await marker, are logged at Warn and post nothing.
  - A cancelled context returns its error.
- `BeamDown.Call` calls `Down`; a failure is logged as a warning and is Done, since Worktree-Teardown ends what is left.

Tests, over fakes of every seam and an injected sleep that records its waits:
- `TestLandRequest`: a write reads back, an unknown key survives a rewrite, an absent file reads as none and removes cleanly.
- `TestBeamUp`: every step runs in order on a fresh pair; a marked pair skips the mark; a failing step halts naming `lyx loom resume`.
- `TestBeamAwait` rows: an accepted request whose re-check passes writes Parent-Align before the marker and the stop, writes the snapshot, removes request and marker and is Done; a request naming another run is removed and the row waits; a failing re-check writes Beam-Await, restarts the dobby, removes marker and request, posts the blockers and waits; a failing restart keeps the marker and posts `lyx beam start`; a failing Parent-Align write keeps the request and accepts it at the next wake; a re-entry with no request and status Parent-Align lifts the hold, removes the marker and posts one interrupted notice; a re-entry with status Beam-Await, no request and the dobby live posts none; a re-entry with status Beam-Await, no request and the dobby down posts one dobby-down notice naming `lyx beam start`; a first entry, with no await marker and the dobby not yet live, reads no `DobbyLive`, posts none and writes the marker; a passing re-check removes the await marker; a re-entry with request and marker standing and a failing re-check leaves the dobby restarted; another machine's ownership removes the request and yields; a failed entry status write is retried at the next wake; a failing Beam-Await write after a failed re-check keeps the request and the marker, posts nothing, and the next wake's accept lifts the hold; a `Recheck` error and a failing `StopDobby` each lift the hold, remove the request and post the error; a failing `Notify` leaves the row waiting; a nil `Watch` wakes on the timer alone; a cancelled context returns its error.
- `TestBeamDown`: a failing stop is Done; `TestNextAwaitWait`: the doubling, the one-minute ceiling and the reset on an event.

`TestMain` runs through `tmuxkit.Main` (`PATTERN-test-isolation`), from its own untagged support file, and every test file carries the `tier0` tag and every test calls `t.Parallel`.
`cmd/lyx`'s `unboundedWaitFiles` gains `internal/pawlengine/beamawait.go`, the file `BeamAwait` and its wait live in, so the daemon-wakeup scan checks its floor; `docs/overview.md`'s module table gains the package.

**ImpactSummary:** a new leaf of the shed family with no caller yet; card 16 registers its producers and card 17 wires their seams, so nothing existing changes behavior here.

**Commit:** 15: pawlengine: the Beam-Up, Beam-Await and Beam-Down producers and the land request
