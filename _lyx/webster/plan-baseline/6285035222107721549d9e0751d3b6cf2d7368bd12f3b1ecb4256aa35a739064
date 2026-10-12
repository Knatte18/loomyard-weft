# Card 3 — shedengine-reroute

**Create:**
- `internal/shedengine#OutcomeReroute`
- `internal/shedengine#rerouteCap`
- `internal/shedengine#rerouteCount`
- `internal/shedengine/reroute_test.go`

**Edit:**
- `internal/shedengine#ProducerDef`
- `internal/shedengine#OutputPointer`
- `internal/shedengine#HistoryEntry`
- `internal/shedengine#Shed`
- `internal/shedengine#Shed.validate`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#episodeStuckCount`
- `internal/shedengine#composeActivity`
- `internal/shedengine#gotoAdmitted`
- `internal/shedengine/doc.go`
- `internal/loomengine#checkCoherence`
- `internal/loomengine#TestCheckCoherence`
- `contracts/specs/loom-status-spec.md`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Give `shedengine` a recipe-declared reroute edge (D9) and a Stuck that halts without routing, both inside its standard-library-only imports (`PATTERN-shed-producer-seam`).

Fields:
- `ProducerDef.OnReroute` names the one row a reroute goes to; empty means the row cannot reroute.
- `OutputPointer.Reroute`, read only on a Stuck verdict, takes the `OnReroute` edge instead of `OnStuck`.
- `OutputPointer.Halt`, read only on a Stuck verdict, blocks the run at its own row with the producer's Reason whatever its `OnStuck`, and the entry is budget-exempt.
  It exists so a row with an `on_stuck` route, PR-Gate, can halt for a human on a lock wait without bouncing; it can only stop a run, never advance or route it.
- `HistoryEntry.RoutedTo` (`routed_to`, omitempty) names a reroute entry's target, and `HistoryEntry.RerouteCapped` (`reroute_capped`, omitempty) marks the Stuck entry of a reroute-cap halt.
  A status file written before either field existed decodes as before.
- `Shed.RerouteCapWayForward` is the told trailing clause of the cap halt; empty falls back to the run verb `runVerb("run", s.RunID)` renders.

`validate` refuses an `OnReroute` naming no producer in the list and one naming the row itself, each with its own message; a cross-segment target is legal.
A Stuck with `Reroute` set on a row whose `OnReroute` is empty is a wiring fault: `stepLocked` writes the run failed with an error naming the row, as it does for an unrecognised outcome.

`stepLocked`'s reroute arm, on a Stuck with `Reroute` set:
- `rerouteCount(history, row)` counts the row's `OutcomeReroute` entries back to its latest Done or its latest `RerouteCapped` entry, whichever is later.
- Below `rerouteCap` it appends `{producer: row, outcome: reroute, routed_to: OnReroute, output, at}`, never spending bounce budget, and persists `current_producer` = `OnReroute` with the state `routedState` gives, so a recorded `pause_after` of the row still fires.
- At the cap it appends a budget-exempt Stuck entry with `RerouteCapped` and persists blocked at the row, with the reason `reroute cap reached: <row> rerouted to <target> <n> times since its last done; <producer reason>; way forward: <RerouteCapWayForward>`.
  The resume re-calls the row, and the capped entry resets the count, so each resume of that halt grants three more reroutes.
- Bound: a row reroutes only to its declared target, only when its producer asks, and at most three times before a human resumes it.

`episodeStuckCount` treats a reroute entry like a goto entry whose target is its `RoutedTo`: it ends the episode of every row sharing the target's segment, or of the target itself when it has none.
`composeActivity` renders a reroute entry as `<producer> → rerouted to <target>`.
`gotoAdmitted`'s history fallback admits a reroute entry's `OnReroute` row as it admits `OnDone` after done.
`PATTERN-shed-verb-set`'s goto clause is unchanged: `OutcomeGoto` stays the goto verb's alone.

`loomengine.checkCoherence` accepts `reroute` as a history-only outcome beside `goto`, with a row in `TestCheckCoherence`; `loom-status-spec.md` adds `reroute` to the outcome list, `routed_to` and `reroute_capped` to the history shape, and the reroute to the budget paragraph.
`refusal-spec.md`'s shed section gains the reroute-cap halt row and the missing-`on_reroute` wiring fault, and `doc.go` documents the reroute edge, the cap and `Halt` beside goto.

`reroute_test.go` pins, through `Step` against a status file: a reroute moves `current_producer` to the target, appends the entry with `routed_to` and leaves the target segment's Stuck count at zero; the fourth reroute since a Done halts blocked with the capped entry and the way-forward clause, and a resumed call reroutes three more times; a Done of the row resets the count; a `Reroute` Stuck on a row with no `OnReroute` writes failed; a `Halt` Stuck on a row with an `OnStuck` blocks at the row with the reason and spends no budget; `validate` rejects an unknown and a self `OnReroute` and accepts a cross-segment one; `composeActivity` renders the reroute wording.

**ImpactSummary:** shedengine's routing gains one optional edge and two optional verdict flags; every existing row and status file behaves as before, and loom's coherence check admits one more history-only outcome.

**Commit:** 3: shedengine: reroute edge with a three-reroute cap, and a halting Stuck
