# Card 31 — batten-pass-wiring

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battencli#markBattenWatched` -> `func markBattenWatched(prime, taskLocation *lyxcwd.Location, scratchDir, producer string, pid int) (bool, error)`
- `internal/battencli#stepMarkWatched_HoldsTheMarkerOnlyWhileItsNoticesReachTheDriversParent`
- `internal/battencli#TestWire_EverySeamFilled`

**Create:**
- `plan:internal/battencli#dobbyReading` -> `func dobbyReading(prime *lyxcwd.Location, slug string) (pid int, sleeps []pauserecord.Interval, err error)`
- `plan:internal/tendercli#ReviveStalled` -> `func ReviveStalled(location *lyxcwd.Location, slug string) error`
- `plan:internal/tendercli#reviveError` -> `func reviveError(report tenderengine.ReviveReport) error`
- `plan:internal/tendercli#TestReviveError` -> `func TestReviveError(t *testing.T)`
- `plan:internal/battencli#TestDobbyReading` -> `func TestDobbyReading(t *testing.T)`

**Uses:**
- `plan:internal/hubgeom#DobbyGeometry`
- `plan:internal/dobbyengine#ReadState`
- `plan:internal/dobbyengine#Live`

**Intent:**
Fill the pass-time seams InnerRun gained, from the dobby that schedules the run (D4, D9).
`dobbyReading` reads the state record of the run's parent worktree's dobby: its pid when it is live (by `dobbyengine.Live`) and the sleeps it recorded; no record, or a dead dobby, reads as this process's pid and no sleeps; an undecodable record reads the same, since card 4's `ReadState` reads it as absent with its own one Warn and `dobbyReading` adds none, rather than returning an error into every pass's wiring; any other read error is returned.
In `wire`, InnerRun's `Sleeps` returns the dobby's recorded sleeps, and `Awake` is the clock every pass shares: the wall time since the Unix epoch less those sleeps (`pauserecord.Excluded`), so the idle watches the memory record restores keep measuring awake time across passes.
`markBattenWatched` takes the pid it writes; `wire`'s marker closure passes the scheduling dobby's pid, so the batten-watched marker stays alive between passes while that dobby lives, and a hand pass with no live dobby writes its own pid, which reads dead once it exits and leaves the child's driver to notify for itself.
InnerRun's `ReviveAPIError` calls `tendercli.ReviveStalled`, which runs `tenderengine.Revive` over the hub's deps for the slug, the conditioned send of tender's revive line to the pair's sessions stalled on an API error, and maps its report through `reviveError`, an error when the report names none sent and a refusal, so the caller's once-per-episode warn sees it.
Tests: `TestDobbyReading` (tier0, temp dirs, parallel) rows: a state record with a dead pid reads this process's pid and no sleeps, a missing record the same, a record whose sleeps are read back, an undecodable record reads as no dobby, this process's pid and no sleeps; a row computes the shared `Awake` reading at two wall times with a recorded sleep between them and finds them apart by the wall gap less the sleep; `TestReviveError` (tier0, parallel) rows: a report with a session sent is no error, one with only a refusal is an error naming it, an empty report is no error; `TestWire_EverySeamFilled` asserts `Sleeps`, `Awake` and `ReviveAPIError` set; the step test of the watched marker passes a pid.

**ImpactSummary:** Batten's wiring fills three InnerRun seams and the watched marker takes its pid from the parent's dobby; tender gains one exported wrapper of its revive.
