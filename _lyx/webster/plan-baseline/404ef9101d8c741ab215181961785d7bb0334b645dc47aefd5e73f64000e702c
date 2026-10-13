# Card 16 — start-resume-go-driver

**Edit:**
- `internal/loomcli#mustSpawnDriver` -> `func mustSpawnDriver(runLockHeld, driverLockHeld, driverStrandLive bool) bool`
- `internal/loomcli#awaitRunLock`
- `internal/loomcli#dispositionForHandshake`
- `internal/loomcli#loomCLI.runDriverSpawnAndWait`
- `internal/loomcli#loomCLI.startCmd`
- `internal/loomcli#loomCLI.resumeFromRunState`
- `internal/loomcli#loomCLI.resumeCmd`
- `internal/loomcli#TestMustSpawnDriver`
- `internal/loomcli#TestRunDriverSpawnAndWait_LiveDriverWithoutParkMarker`
- `internal/loomcli#TestResumeVerb`
- `internal/loomcli#seedGoDriverRun`
- `internal/loomcli#findDriverPIDs`
- `internal/loomcli#waitRunLockFree`
- `internal/loomcli#TestSmokeRunStandalone`
- `internal/loomcli#TestSmokeBootstrapLifecycle`
- `internal/loomcli#TestSmokeStatusStrandAcrossDriverSeeds`
- `internal/loomcli/doc.go`
- `docs/overview.md`
- `pattern/PATTERN-driver-choice-single-site.md`
- `PATTERN.md`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/loomcli#loomCLI.driverLockHeld` -> `func (c *loomCLI) driverLockHeld() (bool, error)`
- `plan:internal/loomcli#TestRunDriverSpawnAndWait_GoArm` -> `func TestRunDriverSpawnAndWait_GoArm(t *testing.T)`

**Uses:**
- `plan:internal/loomcli#spawnDetachedDrive`
- `plan:internal/shedrun#DriverLock`

**Intent:**
D2's liveness at start and resume, and D10: a go-seeded run is driven by `lyx loom drive`, a held driver lock reads as a live driver, and resume spawns a Go driver for a halted go run that has none.

`driverLockHeld` probes `shedrun.DriverLock` without keeping it, creating its directory first, as `runLockHeld` does.
`mustSpawnDriver` spawns only when neither lock is held and no driver strand is live; its doc comment adds the driver lock as the Go driver's liveness between steps.

`runDriverSpawnAndWait`:
- reads `driverLockHeld` beside the run lock and passes it to `mustSpawnDriver`;
- with no spawn needed, a held driver lock and no live strand: a run halted at a hand-back refuses with `shedrun.StartNotParkedKind` through `driverNotParkedMessage`, since its Go driver is still settling; a running run is a no-op success;
- the go arm spawns through `c.spawnGoDriver` (card 15's `spawnDetachedDrive`, which reaps its child) in place of the inline `exec.Command(exe, "loom", "run")`;
- the handshake refusal reads "driver did not take its driver lock".
`startCmd` passes `c.driverLockHeld` as the handshake's `lockHeld`, since `drive` takes the driver lock first and holds it for life.
The doc comments of `awaitRunLock` and `dispositionForHandshake` say the go arm's handshake waits for the driver lock and that the post-run work is `lyx loom drive`'s.
`lyx loom run` stays the generic run body, unchanged and no longer spawned by start.
Batten's awaiting arm, which spawns a decided child through `lyx loom start`, therefore lands on the go arm and spawns `drive`, and the gate reads the recorded decision (D10).

`resumeFromRunState`:
- a halted run whose driver lock is held refuses with `StartNotParkedKind` and a message saying its Go driver is still settling;
- a running run whose driver lock is held is the "already running" success;
- with no live driver strand, no run lock and no driver lock, a halted run (awaiting past `awaitingResumability`, blocked, paused or failed) reads the seed through `shedrun.ReadSeed`.
  A seed recording `shedrun.DriverGo` is checked through `refuseOverUnfinishedMerge`, then gets a Go driver through `c.spawnGoDriver`, and the verb succeeds with `"message": "a Go driver was spawned"` and the pid.
  Every other case keeps `noLiveDriverMessage`.
`resumeCmd`'s `Long` replaces "It never spawns a driver" with: it never spawns an LLM driver, and spawns a Go driver for a halted go-seeded run that has none; its refusal list keeps "lyx loom start" only for a run with no usable driver strand.
Bound: the new spawn needs a halted run, a go seed, no strand, both locks free and no unfinished merge, and the driver it spawns refuses to stack on a held driver lock.

The seed read in resume is a bootstrap verb's read (`PATTERN-driver-choice-single-site`).
The invariant's text widens to admit it, as the decision record's first Constraint states:
- `pattern/PATTERN-driver-choice-single-site.md`'s opening sentence says loom's recorded seed driver is read in its bootstrap verbs `start` and `resume` and in `lyx loom drive`'s own package, each branch selecting only the driving surface;
- `PATTERN.md`'s `PATTERN-driver-choice-single-site` line changes "once per recipe, in its bootstrap verb" to "in its bootstrap verbs (loom's `start` and `resume`)", keeping the rest of the line.

The tripwire `TestDriverChoiceSingleSiteInvariant_OnlyLoomcliReadsTheSeedDriverField` already admits `internal/loomcli`.

Docs:
- `doc.go`'s `# Start` section says the go arm spawns `lyx loom drive` and the handshake waits for the driver lock; its `# Resume` section adds the driver-lock refusal and the Go-driver spawn.
- `docs/overview.md`'s loom bullets on `start` and `lyx loom resume` say the same.
- `refusal-spec.md`'s `start: driver not parked` and `resume: driver not parked` rows add a Go driver holding its driver lock to their triggers.
- Its `resume: no driver strand` row narrows to an llm-seeded or running run.

The `tmux`-tier smoke suite's go-arm tests follow the switch, so Publish's `publish_verify` stays green and reaches no LLM:
- `seedGoDriverRun` also writes the worktree's loom local layer (`configengine.LocalFile`) with `tackler_max_spawns: 0` and `arbiter_max_spawns: 0`, so a go-driven halt such as a failing Preflight hands back with no agent spawned.
- `findDriverPIDs` matches a `loom drive` command line.
- `waitRunLockFree` also waits for the driver lock to read free.
- `TestSmokeRunStandalone` and `TestSmokeBootstrapLifecycle` kill and count `drive` processes, rig a dead go driver through the driver lock where they rig the run lock today, and read the driver log the `drive` process writes.
  Their direct `lyx loom run` subtests are unchanged.
- `TestSmokeStatusStrandAcrossDriverSeeds` keeps its assertions over the go-seeded `start`, now spawning `drive`.

Tests:
- `TestMustSpawnDriver` gains rows: a held driver lock alone spawns nothing, and all three free spawns.
- `TestRunDriverSpawnAndWait_LiveDriverWithoutParkMarker` gains a liveness column: its per-state rows run once with a live strand, as today, and once with the test holding the real driver lock through `lock.TryAcquireWriteLock(shedrun.DriverLock(...))` and no strand.
  Blocked, awaiting, paused and failed refuse with `driver_not_parked`, and running is a no-op, with no spawn.
- `TestRunDriverSpawnAndWait_GoArm` in `start_driver_test.go`, over a fake `spawnGoDriver` whose spawn takes the real driver lock in the test, and the handshake given `c.driverLockHeld`:
  - a go-seeded run with nothing alive calls the spawn seam once and succeeds on the handshake;
  - a spawn seam error refuses and releases the bootstrap lock.
- `TestResumeVerb` gains a `driverLock` column, where the test takes the real driver lock as its `runLock` column does, a `seedDriver` column, and a counting fake `spawnGoDriver`. New rows:
  - a held driver lock over a blocked run refuses with `driver_not_parked`;
  - a held driver lock over a running run is the already-running success;
  - a go-seeded blocked run with no strand spawns once and succeeds;
  - a go-seeded run under an unfinished merge refuses `merge_in_progress` with no spawn;
  - an llm-seeded blocked run with no strand keeps the `no driver strand` refusal;
  - a go-seeded running run with no driver keeps the `no live driver` refusal.
  The table's no-spawn assertion holds on every other row.
`test-budget.yaml`'s `internal/loomcli` row rises by one.

**ImpactSummary:** start's go arm now spawns `lyx loom drive` and waits on the driver lock, resume gains one spawn branch for go-seeded halted runs, the go-seeded tmux smoke tests follow, and `PATTERN-driver-choice-single-site`'s text names resume as a reader; llm-seeded runs and every other branch are unchanged.
