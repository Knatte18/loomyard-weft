# Card 1 — shedengine-block-routing

**Edit:**
- `internal/shedengine#ProducerDef`
- `internal/shedengine#HistoryEntry`
- `internal/shedengine#OutcomeSkipped`
- `internal/shedengine#GotoRequest`
- `internal/shedengine#RelaunchRequest`
- `internal/shedengine#Routing`
- `internal/shedengine#Shed.enter`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#Shed.rerouteStuck`
- `internal/shedengine#Shed.validate`
- `internal/shedengine#Goto`
- `internal/shedengine#Relaunch`
- `internal/shedengine#gotoAdmitted`
- `internal/shedengine/doc.go`
- `internal/shedengine#TestStep_EntryGate`
- `internal/shedengine#TestStep_RerouteMovesToTargetAndEndsTargetSegmentEpisode`
- `internal/shedengine#TestGoto_MovesBlockedRunToTarget`
- `internal/shedengine#TestGoto_BlockedRunAdmitsOnlyEarlierRows`
- `internal/shedengine#TestRelaunch`
- `internal/shedengine#TestShed_Validate`
- `internal/shedengine#TestStatus_JSONRoundTrip`

**Extends:**
- `contracts/specs/loom-status-spec.md`

**Create:**
- `plan:internal/shedengine#ProducerDef.skipTarget` -> `func (d ProducerDef) skipTarget() string`

**Uses:**
- `internal/shedengine#newTestShed`
- `internal/shedengine#commonSeed`
- `internal/shedengine#seedStatus`
- `internal/shedengine#readStatus`
- `internal/shedengine#fixedOutcomeProducer`
- `internal/shedengine#outcomeSequence`
- `internal/shedengine#funcProducer`
- `internal/shedengine#fakeGate`
- `internal/shedengine#tailProducers`

**Intent:** Give the engine the two run-time facts a block needs (D3, D6), with no knowledge of recipes or blocks beyond them.

`ProducerDef` gains two fields:
- `SkipTo string`, the skip target: the row a skipped entry of this row routes to. Empty means the row skips by `OnDone`, so every existing row skips exactly as today.
- `Block string`, the row's block path (outermost block first, joined by `/`), empty for a row outside every block.

`skipTarget` returns `SkipTo` when set and `OnDone` otherwise; it is the one place a skip's route is read.

`HistoryEntry` gains `Block string` with the JSON tag `block,omitempty`, documented like its neighbours: a status file written before the field existed decodes as empty (PATTERN-tolerant-records).
`contracts/specs/loom-status-spec.md`'s history example gains the comment line that an entry of a row inside a block carries `"block": "<block path>"`, the enclosing block rows' names joined by `/`, omitted outside every block.

Readers that take the skip target through `skipTarget`:
- `Shed.enter`: a gate that does not hold appends the skipped entry and routes, persists and looks up the next row by `skipTarget()` where it reads `def.OnDone` today; an empty skip target finishes the run as an empty `OnDone` does now.
- `gotoAdmitted`: a skipped entry's routed row is `def.skipTarget()`; a Done entry keeps `def.OnDone`.
- `Shed.validate` refuses a `SkipTo` naming no producer in the list, in its own message `shedengine: producer %q at index %d has SkipTo %q naming no producer in the list`, beside the OnDone rule.
  Like validate's other rules it is a fail-loud wiring guard in `refusal-spec.md`'s sense, naming no way forward: shedbuild only ever sets a skip target to an existing row (cards 4 and 5), so only a hand-built list reaches it, which `TestShed_Validate` does.

Every history entry carries the row's block path:
- `Shed.stepLocked`'s `appendHistory`, `Shed.enter`'s skipped entry and the three entries `Shed.rerouteStuck` writes set `Block: def.Block`.
- `Goto` and `Relaunch` look the target up in `req.Producers` (already checked by `CheckGotoTarget`) and set `Block` on the goto entry from that definition, so a goto or re-launch entry reads the block from the routing projection the caller passes.

The field docs that list what a projection must carry are updated: `GotoRequest.Producers` and `RelaunchRequest.Producers` now need `Name`, `Segment`, `OnDone`, `OnStuck`, `SkipTo` and `Block`, and the `Routing` type doc names `SkipTo` and `Block` among the fields read.

Bound: `SkipTo` lets a skip jump past rows; only a row whose `EntryGate` does not hold takes it, and validate refuses a target outside the list.
A goto or re-launch moves only a halted run, and only to a row `CheckGotoTarget` admits; a goto onto a gated row bypasses its gate and a re-launch evaluates it, as today.

Update `OutcomeSkipped`'s doc ("routes by the row's OnDone") and `doc.go`'s entry-gate paragraph to say the skip routes by the skip target when set, and describe the two new fields.

The block's goto, re-launch and Stuck behaviour is the engine's existing behaviour on a gated row and stays covered where it is: a goto onto a gated row runs it without evaluating the gate (`TestGoto_StepResumesAtTarget`), a re-launch clears the entered row so the gate is evaluated (`TestRelaunch`), and a Stuck with no `OnStuck` halts at the row and a resume re-calls it (`TestStep_StuckWithNoOnStuck`, `TestRun_ResumeAfterHalt`).
This card adds rows only for what is new, the skip target and the block path.

Tests, all tier0:
- `TestStep_EntryGate` gains rows over a hand-built list shaped like an expanded block (a block row with a `fakeGate`, `OnDone` the first inlined row and `SkipTo` the exit; inlined rows with `Block: "B"`; an inner block row with `Block: "B"` and `SkipTo` the exit; an exit row):
  - a false gate on the block row records one skipped entry with the gate's reason, routes to `SkipTo`, not `OnDone`, and no inlined row is called or recorded;
  - a holding gate: the block row's Done entry carries an empty `Block`, and the next step's entry of the first inlined row carries `Block: "B"`;
  - a false gate on the inner block row, reached by a Done of the inlined row before it, routes to the exit and its skipped entry carries `Block: "B"`.
- `TestStep_RerouteMovesToTargetAndEndsTargetSegmentEpisode` sets `Block` on the rerouting row and asserts the reroute entry carries it.
- `TestGoto_MovesBlockedRunToTarget` asserts the goto entry carries the target's `Block`; `TestRelaunch` gains a row asserting the re-launch entry carries it.
- `TestGoto_BlockedRunAdmitsOnlyEarlierRows` gains a `producers` field, `tailProducers` when nil, and two rows over a block-shaped list: with `current_producer` naming no row and the latest entry a skipped block row, the skip target is admitted and the block row's `OnDone` is not; and with the run blocked at the exit after a pass-over, a goto onto an inlined row of the passed-over block is admitted.
- `TestShed_Validate` gains the `SkipTo` naming no producer row.
- `TestStatus_JSONRoundTrip` asserts `block` round-trips, is omitted when empty, and that a file without the key decodes.

**ImpactSummary:** Additive fields on two engine structs and one routing read changed in the skip and goto admission; every existing row has an empty `SkipTo`, so its skip route and every caller of `shedengine` stay as they are.
