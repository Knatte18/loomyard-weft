# Card 9 — Webster's leftover-strand reclaim stops through shuttle

**Create:**
- `plan:internal/websterengine#StrandStopper` -> `type StrandStopper interface`

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
- `internal/websterengine/strand_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/webstercli/reset_test.go`
- `internal/webstercli/verbs_test.go`
- `internal/websterengine/doc.go`
- `PATTERN.md`

**Uses:**
- `plan:internal/shuttleengine#Runner.StopStrand`

**Intent:**
Move webster's leftover-strand reclaim onto shuttle's stop verb by guid (D7), so one stop path exists, and add D8's `PATTERN-shuttle-stop` entry now that no other package removes a shuttle run's strand.

`StrandStopper` is a one-method seam, `StopStrand(guid string) error`, which `*shuttleengine.Runner` satisfies structurally; it sits in `strand.go` beside the other spawn seams.
`RunDeps`, `BeginDeps` and `RecoverDeps` each gain a `Stopper StrandStopper` field, documented as the seam the reclaim stops a leftover strand through; their `Reed` fields stay for the liveness reads that remain (`StrandLive`).
`reclaimEntryTimeStrands` (called by `Run`) and `RemoveRecoveryStrands` (called by `webstercli`'s reset verb) take the stopper in place of reed; `BeginBatch`'s prior-recovery reclaim, `recoverSpawn`'s and `awaitTerminal`'s strand stops call `deps.Stopper.StopStrand` in place of `removeStrandIfLive`, keeping each site's error wrapping and its Warn context in the caller's log line where the old helper carried it.
`webstercli` fills `Stopper` with `c.runner` in `runDeps`, `beginBatchCmd` and `recoverBatchCmd`, and passes `c.runner` to `RemoveRecoveryStrands`; `loomcli`'s `wire` sets `Stopper: runner` on its `RunDeps`.

`removeStrandIfLive` in `websterengine` is left without callers and deleted by card 10.

`PATTERN.md` gains, under "Agents and prompts", per D8 and the decision added after Discussion naming the driver exception: "`PATTERN-shuttle-stop` — Stopping the strand of a shuttle run from Go outside `internal/shuttleengine`: only through shuttle's stop verb, which settles the run's record before the strand is removed; no other package removes a shuttle run's strand, except `loomcli`'s removal of the loom driver's own strand."
It is review-enforced; no scan test is added, since the remaining raw `RemoveStrand` callers (reed's own verbs, orch, the loom driver) would make one a list of exceptions rather than a check.
`websterengine/doc.go` says the reclaim stops leftover strands through shuttle.

Tests: each websterengine test that reaches a reclaim with a recorded strand gets an in-package recording stopper fake in place of the reed liveness script, asserting the stopped guids (Master's strand and non-terminal recovery strands at entry, the prior recovery strand at begin-batch and recover-batch) and that a stopper error fails the reclaim as a failed removal did; `strand_test.go`'s `RemoveRecoveryStrands` coverage moves to the stopper.
`webstercli`'s reset and verb tests give their CLI fixture a stopper where they reach a reclaim.

**ImpactSummary:** Two `websterengine` functions re-signed with their callers (`Run`, `webstercli` reset) on this card; three deps structs gain a field filled by `webstercli` and `loomcli` wiring, and webster's reclaim now records `stopped` on the run it ends.

**Commit:** 9: webster: reclaim leftover strands through shuttle's stop verb; add PATTERN-shuttle-stop
