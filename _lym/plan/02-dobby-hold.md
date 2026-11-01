# Card 2 — Dobby gives a held run no pass

**Edit:**
- `internal/dobbyengine#RunView`
- `internal/dobbyengine#planPass`
- `internal/dobbyengine#daemon.drivePasses`
- `internal/dobbyengine#TestPlanPass`
- `internal/dobbyengine#TestOwnedElsewhere`
- `internal/dobbyengine/doc.go`
- `internal/battencli#DobbyRuns`
- `internal/beamcli#TestDobbyRuns_PairAtItsOwnAnchor`

**Uses:**
- `plan:internal/pauserecord#HoldCaller`
- `internal/dobbyengine#passHarness.tick`
- `internal/dobbyengine#passHarness.state`
- `internal/dobbyengine#runningRun`
- `internal/beamcli#seedBattenRun`

**Intent:**
D4: dobby gives a run covered by a `HoldCaller` entry no pass of any kind, continue passes included, and its owner-pause recheck owes no lift pass for it.

`RunView` gains `Held bool`: a live entry of the run's own scope carries `pauserecord.HoldCaller`.
`battencli.DobbyRuns` fills it from the coverage it already reads, `record.CoverageOf(slug, now).Held`.

`planPass` returns `passNone` for a run whose `Held` is set, right after its running, cancel-marker and in-flight checks and before the owned-elsewhere branch, so neither a continue pass under the cover nor the owned-elsewhere probe pass runs.
The lift of the hold uncovers the run, so the next tick's lift trigger gives it the full pass as for any lifted pause; a `HoldCaller` entry lifted by hand (`lym tender resume`) is a lift like any other.

`daemon.drivePasses` calls `recheckOwnerPause` only for a run that is not `Held`: under a hold no owner entry stands to lift, and the recheck's `liftPending` mark would otherwise owe the run a lift pass for an entry that was never lifted.
Bound: this skips only the recheck; a held run whose owner reads another machine once the hold lifts gets the owned-elsewhere handling as today.
Batten's first pass refusing a covered run with `PassExitCovered` stays the backstop.

Tests:
- `TestPlanPass` gains rows: a held running run at Run-Shed with a child write gets none; a held run with a lift trigger still covered by the hold gets none; a held run recorded owned elsewhere with a lift trigger, still covered by the hold, gets none, which the owned-elsewhere branch alone would answer with a full pass.
- `TestOwnedElsewhere` gains a subtest: a run recorded owned elsewhere whose view reads `Held`, with the owner reading this machine, on a probe tick calls no `LiftOwnerPause`, records no lift pending and spawns no pass.
- `TestDobbyRuns_PairAtItsOwnAnchor` gains an assertion: with a live `HoldCaller` entry of the run's scope in the pause record, the run's view reads `Held` and `Covered`; with an `OwnerCaller` entry it reads `Covered` and not `Held`.

`doc.go`'s passes section states that a run a hold-caller entry covers gets no pass of any kind and no owner-pause recheck, and that the lift of the hold gives it the full pass.

**ImpactSummary:** planPass gains one early none, drivePasses skips one recheck, DobbyRuns sets one new field; one dobby package and its one reader in battencli.

**Commit:** 2: dobby gives a run an adopt holds no pass
