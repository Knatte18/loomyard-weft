# Card 1 — gated-turn-end-holds

**Edit:**
- `internal/shuttleengine#Run.expiredTurnEnd`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Spec`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/waiting_test.go`
- `internal/shuttleengine/claudeengine/testdata/corpus/master-fork-running/case.yaml`

**Delete:**
- `internal/shuttleengine#Run.countsAtOnce`

**Uses:**
- `internal/shuttleengine#Run.classifyDeadlineExpiry`
- `internal/shuttleengine#Run.endedShells`
- `internal/shuttleengine/completionsignal_enforcement_test.go`

**Intent:** Remove the rule that counts a gated run's waiting turn end as a Done arrival at once (decision D1).
In `expiredTurnEnd`, after the past-bound logging, the turn end counts at once only when the run is ungated and every output file exists; a gated run's waiting turn end with any task outstanding, a shell of either signal or a fork, returns no outcome and keeps waiting, exactly as a turn end with an output file missing does today.
Delete `countsAtOnce`, its only caller being `expiredTurnEnd`.
With it gone, `Run.startedFresh` and `Run.gatedArrival` have no reader: drop both fields from the `Run` struct, the `startedFresh` initializer in `Runner.start`, and the `gatedArrival` assignment at the gated Done branch of `Wait`.
Keep `countedTasks`, which the ungated at-once count still fills for `endedShells`.
Keep `awaitedShell`, `awaitedLabel` and `Spec.AwaitedShellPrefixes`: the wait marker's `shellWaitActive` still reads them, so reword the field's doc on `Spec` to say the prefixes only keep a shell out of the background-shell wait marker, since every shell now holds a gated turn end alike.
Update the doc comments that state the old rule: the `wait.go` file doc line "A gated run whose output files all exist, started fresh, with no gated arrival yet and only unawaited shells outstanding, finishes Done at once", the `expiredTurnEnd` doc, the `Run` field docs, and the "A gated run's waiting turn end with every output file present is not an arrival" paragraph of `doc.go`.
The new wording: a gated run's waiting turn end is never an arrival; it waits until the task's completion starts the next turn, bounded by the run deadline and the liveness check; at the deadline `classifyDeadlineExpiry` ends the run `OutcomeDone` when the output files exist, finalize evaluates the gate once more over the tree as it stands, and the shell lands in `Result.EndedShells` with its result never read by the run.
Say in `doc.go` that `Result.EndedShells` records shells whose result the run never read.
No new negative verdict is added and no `allOutputFilesExist` call site changes, so the audited sets in `completionsignal_enforcement_test.go` stay as they are; run that test to confirm.

Tests, in `waiting_test.go`:
rework `TestWait_GatedFreshRunWithOutstandingShellEvaluatesGateAtOnce` into a test that pins the hold: a gated run with every output file present and one unawaited shell outstanding stays waiting through the turn end, the gate is not evaluated, and a later `STOP` line with nothing outstanding is the arrival that runs the gate once, finishes `OutcomeDone` and records no ended shell;
drop its `startedFresh` and `gatedArrival` uses.
Add a row to `TestWait_RecordsEndedShells`: a gated run with the output file present, the transcript shell outstanding and a jumping clock reaches the run deadline, ends `OutcomeDone`, has its gate evaluated exactly once at finalize, and records the shell in `EndedShells` with its time outstanding.
Keep the row "an ungated at-once finish", which pins that an ungated run under the same facts still counts at once, and the awaited-shell row.
Drop every `run.startedFresh = ...` assignment in the file.

In the corpus case, update the `what_went_wrong` and `old_reading` commentary of the last turn end: it says the turn end stays waiting on the shell whether or not Master's output files exist, and that only the run deadline, the liveness check or a stop ends such a run.
The replayed `reading`, `outputs_exist`, `tasks` and `states` fields are unchanged.

**ImpactSummary:** One early exit removed from shuttle's wait loop plus two dead fields; only gated producers change behavior, websterengine's own wording follows on card 2.
