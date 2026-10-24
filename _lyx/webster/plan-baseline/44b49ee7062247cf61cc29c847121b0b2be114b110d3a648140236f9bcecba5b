# Card 17 — shed-entry-gate-engine

**Create:**
- `plan:internal/shedengine#OutcomeSkipped` -> `const OutcomeSkipped Outcome = "skipped"`
- `plan:internal/shedengine#EntryGate` -> `type EntryGate func() (holds bool, reason string, err error)`
- `plan:internal/shedengine#Shed.enter` -> `func (s *Shed) enter(st Status, def ProducerDef) (ProducerDef, *StepResult, error)`
- `plan:internal/shedengine#Shed.persistRouted` -> `func (s *Shed) persistRouted(nextCurrentProducer string, nextState State, nextHistory []HistoryEntry, clearConditions bool, routedTo, pauseReason string) error`
- `plan:internal/shedengine#TestStep_EntryGate` -> `func TestStep_EntryGate(t *testing.T)`

**Edit:**
- `internal/shedengine#ProducerDef`
- `internal/shedengine#Status`
- `internal/shedengine#HistoryEntry`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#Shed.persistMarked` -> `func (s *Shed) persistMarked(markResume bool, nextCurrentProducer string, nextState State, nextError string, nextHistory []HistoryEntry, consumePause, clearConditions bool, routedTo string, transient TransientClass, parentNotice, pauseReason string, clearEntered bool) error`
- `internal/shedengine#Shed.persistTransient`
- `internal/shedengine#Shed.rerouteStuck`
- `internal/shedengine#TestGoto_StepResumesAtTarget`
- `internal/shedengine#Goto`
- `internal/shedengine#Relaunch`
- `internal/shedengine#RenameProducers`
- `internal/shedengine#composeActivity`
- `internal/shedengine#episodeStuckCount`
- `internal/shedengine#gotoAdmitted`
- `internal/shedengine#TestGoto_MovesBlockedRunToTarget`
- `internal/shedengine#TestRelaunch`
- `internal/shedengine#TestRenameProducers`
- `internal/shedengine#TestComposeActivity`
- `internal/shedengine#TestStatus_JSONRoundTrip`

**Extends:**
- `internal/shedengine/doc.go`

**Intent:**
Shed evaluates a row's entry gate when the run enters the row (D8); the gate reaches `shedengine` as a function on `ProducerDef`, and the package imports nothing new (`PATTERN-shed-producer-seam`).

`ProducerDef.EntryGate` is an `EntryGate`, nil for an ungated row; it answers whether the row runs, the check's text with the values it read, and an evaluation error.
`Status.EnteredRow` (`entered_row`, omitted when empty) names the row the run entered; a status without the field, or naming another row, reads as not entered (`PATTERN-tolerant-records`).
`HistoryEntry.Reason` (`reason`, omitted when empty) carries a skip's text.

`Shed.stepLocked` calls `Shed.enter` after the pause checks and before the resume write.
`enter` evaluates a gated row whose name `EnteredRow` does not hold: a holding gate persists the row as entered and returns it to call; a false gate appends an `OutcomeSkipped` entry with the reason, routes by the row's `OnDone` as a Done does, spending no bounce budget, and moves on to that row in the same step, so consecutive skips chain.
At each row of a chain a `pause_before` naming it pauses the run before its gate is evaluated, so a chain stops at a paused row and a setting changed during the pause decides the gate; a `pause_after` naming a skipped row pauses the run at the row the skip routes to, with the reason `paused after <row>, as requested`, clearing both conditions as a fired condition does; an empty `OnDone` finishes the run done.
A chain longer than the producer list, a wiring fault no shipped recipe reaches, stops the step as an evaluation error does: the status is written failed naming the rows of the chain and the way forward of fixing the entry gates' settings or moving the run with `lyx shed goto`, then resuming.
An evaluation error leaves the row not entered and stops the step as a producer error does: the status is written failed before the step returns (`PATTERN-failure-stop-visible`), so a resume evaluates again.

`persistMarked` takes an explicit `clearEntered` argument that clears `EnteredRow` in the same write; `persistTransient` and the resume write in `stepLocked` pass false, and the new `persistRouted`, which writes as `persistTransient` does with an empty error, transient class and notice, passes true.
`persistRouted` replaces `persistTransient` at every routed call site, the Done route and the bounced Stuck in `stepLocked` and the reroute in `rerouteStuck`, and writes every skip, so a row is entered only between its own entry evaluation and its verdict, and a route back into the entered row, a direct self-route included, through skipped rows or not, evaluates its gate.
A resume or a halt re-calls the entered row without evaluating it, since those writes keep the field.
A skip write carries a pending caller label through untouched rather than handing it to the `skipped` entry, so the label of a resumed run lands on the next verdict, never on a skip.
`Goto` names its target as entered, so every goto through the shed goto verb, whoever calls it, runs the target without evaluation; `Relaunch` with a target it moves to clears the field, so the target is evaluated, while one without a target, or with the target already in effect, writes the status unchanged and the entered row is re-called; `RenameProducers` renames the entered row.
`composeActivity` reads a skip as `<row> → skipped`; `episodeStuckCount` treats a skipped entry as a Done by its row, ending the episode as a Done does; `gotoAdmitted` reads a skipped entry's routed row as its `OnDone`.

Bound: a gate skips only its own row, and the shed goto verb is the one bypass.

`TestStep_EntryGate` drives a Shed with fake gates: a false gate records `skipped` with its reason, follows `on_done` and spends no budget; a chain of two skips runs in one step; a skip of a row whose `on_done` is empty finishes the run done; a chain longer than the list writes failed naming the rows; an evaluation error writes failed, leaves the row not entered and a resume evaluates again; a resume of an entered row re-calls it without evaluating; a Stuck routed back to its own row through a skipped row evaluates that row's gate, and so does a gated row whose `on_stuck` names itself; a `pause_before` on a gated row halts before evaluation and a gate answer changed during the pause decides; a `pause_after` on a skipped row pauses at its `on_done` row; a run resumed from blocked with a `ResumedBy` label whose first row is skipped puts the label on the next verdict's entry, not the `skipped` entry.
`TestGoto_MovesBlockedRunToTarget` asserts the goto names its target in `entered_row`, and `TestGoto_StepResumesAtTarget` steps after a goto onto a gated-off row and asserts the row is called; `TestRelaunch` asserts a target relaunch writes the status with `entered_row` cleared and an untargeted or in-effect one leaves it as it stood; `TestRenameProducers`, `TestComposeActivity` and `TestStatus_JSONRoundTrip` cover the field, the activity text and an old status without it.

**ImpactSummary:** Adds the entry gate to the Shed loop and status; every shed run reads the new field, and no ungated row's behavior changes.
