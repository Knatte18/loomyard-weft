# Card 9 — Webster's leftover-strand reclaim stops through shuttle

**Create:**
- `internal/websterengine#StrandStopper`
- `internal/testkit/shedfake#strandStopper`
- `internal/websterengine/stopper_test.go`

**Delete:**
- `internal/testkit/shedfake#reedOps`

**Edit:**
- `internal/websterengine#RunDeps`
- `internal/websterengine#BeginDeps`
- `internal/websterengine#RecoverDeps`
- `internal/websterengine#reclaimEntryTimeStrands` -> `func reclaimEntryTimeStrands(stopper StrandStopper, st *State) error`
- `internal/websterengine#RemoveRecoveryStrands` -> `func RemoveRecoveryStrands(stopper StrandStopper, st *State) error`
- `internal/websterengine#Run`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine#awaitTerminal`
- `internal/webstercli#websterCLI.runDeps`
- `internal/webstercli#websterCLI.beginBatchCmd`
- `internal/webstercli#websterCLI.recoverBatchCmd`
- `internal/webstercli#websterCLI.resetCmd`
- `internal/loomcli#loomCLI.wire`
- `internal/shedrecipe#websterEntry`
- `internal/testkit/shedfake#WebsterSeams`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/webstercli/reset_test.go`
- `internal/webstercli/verbs_test.go`
- `internal/shedrecipe/entries_simple_test.go`
- `internal/testkit/shedfake/shedfake_test.go`
- `internal/battencli/wire_seams_test.go`
- `internal/loomcli/wiring_test.go`
- `internal/websterengine/doc.go`
- `PATTERN.md`

**Uses:**
- `internal/shuttleengine#Runner.StopStrand`

**Intent:**
Move webster's leftover-strand reclaim onto shuttle's stop verb by guid (D7), so one stop path exists, and add D8's `PATTERN-shuttle-stop` entry now that no other package removes a shuttle run's strand but the two the entry names as exceptions.

`StrandStopper` is a one-method seam, `StopStrand(guid string) error`, which `*shuttleengine.Runner` satisfies structurally; it sits in `strand.go` beside the other spawn seams.
`RunDeps`, `BeginDeps` and `RecoverDeps` each gain a `Stopper StrandStopper` field, documented as the seam the reclaim stops a leftover strand through.
`RunDeps.Reed` and `BeginDeps.Reed` are deleted, since the reclaims this card moves were their only readers; only `RecoverDeps.Reed` stays, for `awaitTerminal`'s `StrandLive` read.
Their fills go: `webstercli`'s `runDeps` and `beginBatchCmd`, `loomcli`'s `wire`, and the `runlevel_test.go` and `beginbatch_test.go` fixtures.
The Webster row's seam check in `shedrecipe`'s `websterEntry` requires `WebsterDeps.Stopper` in place of `WebsterDeps.Reed`, so a nil stopper is refused at construction rather than panicking at a reclaim.
`shedfake.WebsterSeams` fills `Stopper` with a `strandStopper` placeholder (an embedded `websterengine.StrandStopper`, like its siblings) in place of `Reed`, whose `reedOps` placeholder goes.
The tests naming the seam follow: `entries_simple_test.go`'s seam-field lists and nil-seam row, `shedfake_test.go`'s `WebsterSeams` assertion, `battencli`'s `intentionallyNil` row and `loomcli`'s "webster deps are fully populated" subtest each name `Stopper` in place of `Reed`.
`reclaimEntryTimeStrands` (called by `Run`) and `RemoveRecoveryStrands` (called by `webstercli`'s reset verb) take the stopper in place of reed; `BeginBatch`'s prior-recovery reclaim, `recoverSpawn`'s and `awaitTerminal`'s strand stops call `deps.Stopper.StopStrand` in place of `removeStrandIfLive`, keeping each site's error wrapping and its Warn context in the caller's log line where the old helper carried it.
`webstercli` fills `Stopper` with `c.runner` in `runDeps`, `beginBatchCmd` and `recoverBatchCmd`, and passes `c.runner` to `RemoveRecoveryStrands`; `loomcli`'s `wire` sets `Stopper: runner` on its `RunDeps`.

Bound (this card routes webster's reclaims onto a verb that writes a record before it removes a strand): `StopStrand` can now let two things through that the raw removal could not.
A `run.json` write failure removes nothing, so the strand stays live behind a record reading `running`; at the entry, begin-batch and recover-spawn reclaims that error fails the reclaim exactly as a failed removal does today, so no replacement is spawned beside the live strand.
`awaitTerminal` keeps treating a stop error as a warning, as it treats a failed removal today: in both cases the strand stays live and the warning names its guid, which is no wider than today's residual.
The batch's terminal digest is already on disk, so refusing would not stop that strand either; what bounds it is the warning's guid for `lyx reed remove`, and webster's reset, whose `RemoveRecoveryStrands` stops every recorded recovery strand.
A removal failure after the write leaves a `stopped` record over a live strand, reported as the same error, which attach reads as respawn-eligible and stops as superseded (card 7).
A guid with no run record is removed as today, and a record already terminal or whose output files all exist keeps its outcome.

`removeStrandIfLive` in `websterengine` is left without callers and deleted by card 10.

`PATTERN.md` gains, under "Agents and prompts", per D8, the decision added after Discussion naming the driver exception, and the parent's round-2 decision naming the orch exception: "`PATTERN-shuttle-stop` — Stopping the strand of a shuttle run from Go outside `internal/shuttleengine`: only through shuttle's stop verb, which settles the run's record before the strand is removed; no other package removes a shuttle run's strand, except `loomcli`'s removal of the loom driver's own strand and `orchcli`'s removal of the orch strand."
The orch exception follows the driver's reasoning.
The orch session is a shuttle run: `orchcli` starts it through `Runner.Start`, so it has a run record.
`orchcli`'s `stop` and relaunch remove its strand raw, and moving them onto the stop verb is outside this task's scope.
No attach probe or `Wait` reads the orch record, since `orchcli` keeps only the strand guid from `Start`, so the bug class of #444 does not reach it.
Without the named exception the entry would be false the day it lands.
It is review-enforced; no scan test is added, since the remaining raw `RemoveStrand` callers (reed's own verbs, orch, the loom driver) would make one a list of exceptions rather than a check.
`websterengine/doc.go` says the reclaim stops leftover strands through shuttle.

Tests, one existing test per reclaim site, each given the recording stopper in place of the reed liveness script:

- a new `stopper_test.go` holds the recording `StrandStopper` double the reclaim tests share and a new `TestRemoveRecoveryStrands`, since no test pinned that function: it tables which recorded strands are stopped, in batch-number order, and that the first stop failure is returned with its guid.
- `runlevel_test.go`: `TestRun_EntryHousekeeping`'s rows assert Master's strand and the non-terminal recovery strands stopped at entry, and none on a fresh run.
- `beginbatch_test.go`: `TestBeginBatch_ReclaimsPriorRecoveryStrandBeforeOverwrite` asserts the prior recovery strand stopped before the record overwrite.
- `recoverbatch_test.go`: `TestRecoverBatch_FirstCallSpawnsArchivesStaleReportAndStopsLiveStrand` asserts the live strand stopped, and `TestRecoverSpawnOrAttach` gains one row where a stopper error fails the reclaim with no replacement spawned.
- `webstercli`'s `reset_test.go` and `verbs_test.go` give their CLI fixture a stopper where they reach a reclaim, with no new test or row.

**ImpactSummary:** Two `websterengine` functions re-signed with their callers (`Run`, `webstercli` reset) on this card; three deps structs gain a field filled by `webstercli` and `loomcli` wiring and checked by the Webster row, two lose their `Reed` field, and webster's reclaim now records `stopped` on the run it ends.

**Commit:** 9: webster: reclaim leftover strands through shuttle's stop verb; add PATTERN-shuttle-stop
