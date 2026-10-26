# Card 23 — dobby-land-notice-and-slots

**Create:**
- `plan:internal/dobbyengine#LandReading` -> `type LandReading struct`
- `plan:internal/dobbyengine#LandNoticeState` -> `type LandNoticeState struct`
- `plan:internal/dobbyengine#daemon.landNotice` -> `func (d *daemon) landNotice(ctx context.Context) error`
- `plan:internal/dobbyengine#TestLandNotice` -> `func TestLandNotice(t *testing.T)`
- `plan:internal/beamcli#beamLandReading` -> `func beamLandReading(prime *lyxcwd.Location, slug string) (dobbyengine.LandReading, error)`
- `plan:internal/battencli#runTakesNoSlot` -> `func runTakesNoSlot(task boardengine.Task) bool`
- `plan:internal/battencli#TestRunTakesNoSlot` -> `func TestRunTakesNoSlot(t *testing.T)`

**Edit:**
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#State`
- `internal/dobbyengine#RunView`
- `internal/dobbyengine#daemon.tick`
- `internal/dobbyengine#slotsTaken`
- `internal/dobbyengine#TestStartReady_RefusalsAndBackoff`
- `internal/dobbyengine#TestBuildView`
- `internal/battencli#DobbyRuns`
- `internal/beamcli#hubDobbyDeps`
- `internal/dobbyengine/doc.go`
- `internal/beamcli/doc.go`

**Intent:**
D12's "beam can land" notice and D9's slot rule for a pawl run under its parent's dobby.

- `Deps` gains `CanLand func() (LandReading, error)`; `LandReading` holds `Passes`, whether `BeamLandRefusal`'s blockers are empty, and the parent's owner and adopt subject when another machine owns the beam's parent.
  `daemon.tick` calls `landNotice` after the scheduling steps on every tick of a beam's dobby, the prime's posting none and a nil `CanLand` checking nothing; the tick already runs on every board change it watches and on every pass exit, so a member's teardown reaching done, a cancel with `--remove`, a member moved off the project and one marked done by hand each trigger it.
  When the reading passes and the state record's `LandNotice` holds no sent episode, it posts its own orch one notice naming `lyx beam land <slug>`, or, when another machine owns the parent, that it does and `lyx adopt <subject>` (`lyx adopt root`) instead, and records the episode as sent; a reading that does not pass clears the episode, so a member opened again re-arms it; a failed read is logged once per episode and changes nothing.
  `State.LandNotice` keeps the sent episode, so a restarted dobby sends no second notice; a failed post leaves it unsent for the next tick.
- `RunView` gains `NoSlot`, true for a pawl run whose landing hold does not stand; `slotsTaken` counts no such run, so a pawl run takes no slot under `max_runs` while it holds no hold and takes one from Parent-Align on, and the parent dobby still schedules its passes, owed full passes and probes as for every run it drives; the view's `max_runs` reasons follow, since `BuildView` counts through `slotsTaken`.
- `battencli.DobbyRuns` fills `NoSlot` from the run's board entry through `runTakesNoSlot`: a beam project whose status `IsRunStatus` accepts and `RunHolds` does not.
- `hubDobbyDeps` fills `CanLand` with `beamLandReading`, over `battencli.ReadBeamLandBlockers`, the parent through `hubgeom.ParentOf` and its owner and subject through `Owners.Effective` and `Owners.AdoptSubject`, for a beam's dobby only.

`TestLandNotice` ticks a beam's dobby over a fake `CanLand`: a passing reading posts one notice naming `lyx beam land <slug>` and a second tick posts none; a parent owned elsewhere names its owner and `lyx adopt <subject>`; a failing reading then a passing one posts a second notice; a restart over a record holding the sent episode posts none; the prime's dobby posts none; a failed post retries on the next tick.
`TestStartReady_RefusalsAndBackoff` gains a dobby with `max_runs: 1` driving a pawl run with `NoSlot` that still starts a ready task, and one driving it without `NoSlot` that does not; `TestBuildView` gains the same slot reading; `TestRunTakesNoSlot`, a tier0 table, reads no slot for a beam project at Beam-Await and at Worktree-Create, and a slot for one at Parent-Align, for a task and for an entry with no run status.
The docs describe the notice and the slot rule.

**ImpactSummary:** one optional dobby seam, one state record key and one run-view field, read only by beam dobbies and the slot count; task runs take slots as before.

**Commit:** 23: A beam's dobby posts one "can land" notice; a pawl run without its hold takes no slot
