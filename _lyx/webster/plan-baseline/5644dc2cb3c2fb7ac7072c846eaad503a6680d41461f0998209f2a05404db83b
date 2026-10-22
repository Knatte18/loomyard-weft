# Card 13 — An owned-elsewhere pause ends the run's processes on the machine it left

**Create:**
- `plan:internal/sheddriver#RestepRecorded` -> `func RestepRecorded(records []shedrun.SettleRecord, status shedengine.Status) bool`
- `plan:internal/sheddriver#TestRestepRecorded` -> `func TestRestepRecorded(t *testing.T)`
- `plan:internal/battenshed#FinalizePastMerge` -> `func FinalizePastMerge(status shedengine.Status, restepped func(shedengine.Status) (bool, error), parentHolds func() (bool, error)) (bool, error)`
- `plan:internal/tendercli#EndOwnedElsewhere` -> `func EndOwnedElsewhere(location *lyxcwd.Location, slug string) ([]runproc.Ended, error)`
- `plan:internal/tendercli#ownedElsewhereEnder` -> `type ownedElsewhereEnder struct`
- `plan:internal/tendercli#ownedElsewhereEnder.run` -> `func (e ownedElsewhereEnder) run() ([]runproc.Ended, error)`
- `plan:internal/tendercli#TestOwnedElsewhereEnder` -> `func TestOwnedElsewhereEnder(t *testing.T)`
- `plan:internal/tendercli#endOwnedElsewhere` -> `var endOwnedElsewhere = EndOwnedElsewhere`

**Delete:**
- `internal/battencli#childRestepped`
- `internal/battencli#TestChildRestepped`

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battenshed#innerRunProducer.keepsLockForFinalize`
- `internal/tendercli#PauseOwnedElsewhere`
- `internal/hibernateengine#Deps`
- `internal/hibernateengine#holdRuns`
- `internal/hibernateengine#fakeWorld`
- `internal/hibernateengine#fakeWorld.deps`
- `internal/hibernateengine#TestWatch`
- `internal/hibernatecli#hubDeps`
- `internal/tendercli#TestPauseOwnedElsewhere_PausesTheSlugOnceUnderTheOwnerCaller`
- `internal/battenshed#TestInnerRun_OwnerCheck`
- `internal/sheddriver/doc.go`
- `internal/tendercli/doc.go`
- `internal/hibernateengine/doc.go`
- `internal/battenshed/doc.go`

**Uses:**
- `internal/runproc#End`
- `internal/runproc#SurvivorsError`
- `plan:internal/fabricengine#RunProcDir`
- `internal/sheddriver#StopKeyOf`
- `internal/fabricengine#ParentHoldsTaskWork`

**Intent:**
D7: every site where an owned-elsewhere verdict pauses a run also ends the run's owned processes on this machine, so a driver chain parked under the owner pause never outlives the adoption.
The three sites are batten's owner probe (`innerRunProducer.ownedElsewhere`, through its `PauseOwned` seam), `lyx orch start`'s `pauseOwnedRuns` (through the installed owner pause) and the hibernate wake's `holdRuns`.
The first two already call `tendercli.PauseOwnedElsewhere`, so the end lives there; the wake pauses through `tenderengine.Pause` and gains a seam.

Finalize exception.
A run whose Finalize is past its parent-side merge keeps its parent lock and finishes its landing (`PATTERN-owner-machine`), so it is not ended at any of the three sites; its processes are ended by the pair's removal.
`FinalizePastMerge` is `keepsLockForFinalize`'s decision with its seams as arguments: Finalize running, or failed on a transient cause with no re-step recorded, while the parent holds the task's work; it returns the seam's error when a read fails.
`keepsLockForFinalize` calls it and keeps its own episode bookkeeping, holding the lock on an error as today, so `TestInnerRun_OwnerCheck` keeps covering it unchanged.
`RestepRecorded` moves `childRestepped` into `sheddriver`, beside `StopKeyOf` and the settle records it reads, so `tendercli` can read a re-step without importing `battencli`, which imports `tendercli`; `battenCLI.wire`'s `Restepped` seam calls it, and `TestRestepRecorded` takes `TestChildRestepped`'s rows.

The end.
`EndOwnedElsewhere` resolves the hub's prime and the slug's task worktree and runs `ownedElsewhereEnder` with production seams: the task worktree's shed status, `sheddriver.ReadSettleRecords` with `RestepRecorded`, `fabricengine.ParentHoldsTaskWork`, and `runproc.End(RunProcDir(hub), slug, os.Getpid())`.
`run` ends nothing when `FinalizePastMerge` reads true or fails, and otherwise calls the end; a task worktree with no status file, and a task worktree that is gone, are ended by slug, since no Finalize can be running there.
Inside `lyx batten run` the caller is batten, so `End` spares batten and its ancestors and ends the driver chain and the pair's pushes; batten keeps its owner hold and probe, adds no restart, and the pair's removal (`fabric remove` or `reed down`, which read the run owned elsewhere or paused) ends it.
From `lyx orch start` and the wake, batten is ended with the chain; `lyx adopt`'s step 5 (`battencli.ResumeInWindow`) later starts a batten that spawns the chain again, the driver lock refusing a second driver.
`PauseOwnedElsewhere` calls `EndOwnedElsewhere`, through the package var `endOwnedElsewhere` so its test swaps it, after a successful pause, logs the ended processes at Info and an error or survivors at Warn, and returns nil for them: the pause stands.
`ownedElsewhereEnder.run` wraps an end error and a `*runproc.SurvivorsError` with the way forward, `lyx fabric remove <slug>`, or `lyx reed down` in the pair, while the pair exists, and `lyx tender reap` once it is gone, so each Warn line that logs it names that step.
`holdRuns` calls the new `Deps.EndOwned func(slug string) error` after a successful owner pause (`run.Owner` set) and logs its error at Warn, the notice unchanged; nil skips it; `hubDeps` fills it with `EndOwnedElsewhere` over the prime.

Tests.
`TestOwnedElsewhereEnder` rows over fake seams: a status off Finalize ends; Finalize running with the parent holding the work does not end; failed transient without a re-step and holding does not; failed transient with a re-step ends; a status read error, a re-step read error and a parent read error end nothing; no status file ends; a gone task worktree ends; a `*runproc.SurvivorsError` from the end is returned, still matching `errors.As`, and it and an end error each name `lyx fabric remove slug` and `lyx tender reap`.
`TestPauseOwnedElsewhere_PausesTheSlugOnceUnderTheOwnerCaller` swaps `endOwnedElsewhere` for a recorder and asserts it is called once with the slug after the pause entry is written, and that a recorder returning an error, and one returning a `*runproc.SurvivorsError`, still leave `PauseOwnedElsewhere` returning nil with the entry standing.
`TestWatch` gains rows through `fakeWorld`: a wake whose reconcile reads a run owned elsewhere pauses it with the owner caller and then calls `EndOwned` for that slug; a stopped run's pause calls none; an owner pause that fails calls none; an `EndOwned` error leaves the notice as it was.
`TestInnerRun_OwnerCheck` gains a row for the restart D7 relies on at the owner-probe site: after an owned-elsewhere verdict paused the run and the child's driver holds neither driver lock nor strand, the harness's later owned-here verdict with the owner pause lifted yields the driver-dead notice once, and none while the owner pause covers the run.
The other two restart readings already have rows: `lyx loom resume` refusing a running child with no live driver, naming `lyx loom start` (`internal/loomcli#TestResumeVerb`, "running with no live driver is refused"), and batten spawning a running child whose driver strand is dead with the driver lock free (`internal/battenshed/innerrun_test.go`, "DeadStrandWithAFreeLockSpawns"), which is what the batten `lyx adopt` resumes does.
The `tendercli`, `hibernateengine` and `battenshed` docs name the end, its Finalize exception and that batten survives its own probe's end; the `sheddriver` doc names `RestepRecorded` beside the settle records.

**ImpactSummary:** one function moves from battencli to sheddriver with its single caller and test; PauseOwnedElsewhere and holdRuns gain an end after the pause; keepsLockForFinalize's behaviour is unchanged.

**Commit:** 13: an owned-elsewhere pause ends the run's processes on this machine
