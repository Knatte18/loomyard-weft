# Card 5 — seat-chair-hold-deferral

**Edit:**
- `internal/seatengine#Handle`
- `internal/seatengine#LiveTable`
- `internal/seatengine#Engine.Run`
- `internal/seatengine#Engine.startSeat` -> `func (e *Engine) startSeat(table Table, seat Seat, names map[string]string, work *advisorWork) (Handle, error)`
- `internal/seatengine#Engine.Probe`
- `internal/seatengine#Engine.probeSeat` -> `func (e *Engine) probeSeat(table Table, seat Seat, gated bool, holdDeferral func() bool) (Handle, bool, error)`
- `internal/seatengine#Engine.Resume`
- `internal/seatengine/seatfake_test.go`
- `internal/seatengine/engine_test.go`
- `internal/seatengine/smoke_seats_test.go`
- `internal/seatengine/testdata/smoke-advisor.md`
- `internal/seatengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/seatengine#advisorWork` -> `type advisorWork struct`
- `plan:internal/seatengine#advisorWork.add` -> `func (w *advisorWork) add(run *seatRun)`
- `plan:internal/seatengine#advisorWork.bindSender` -> `func (w *advisorWork) bindSender(sender *noticeSender)`
- `plan:internal/seatengine#advisorWork.outstanding` -> `func (w *advisorWork) outstanding() bool`

**Uses:**
- `plan:internal/shuttleengine#Run.SessionState`
- `internal/seatengine#noticeSender.wasSent`
- `internal/seatengine#advisorUpNotice`

**Intent:**
Defer the chair's held-turn notice while advisor work is outstanding (decision D10), so a chair's planned wait on its advisors stops reaching the parent as a stall notice.

`Handle` gains `SessionState() (shuttleengine.RunSessionState, error)`, the advisor run's own session-state reading; `*shuttleengine.Run` satisfies it through card 2.

`advisorWork` holds, under a mutex, the advisors a table run has started and the notice sender.
`outstanding` reports true while any held advisor whose run has not ended (its watch goroutine has not returned) either has an up line the sender has not landed (`wasSent` of its `advisorUpNotice` is false) or reads mid-turn: its `SessionState` names `SessionBusy` with cause `SessionCauseTurn`, a turn start with no turn end after it.
An advisor whose run has ended never counts, a timed-out one included, though shuttle keeps a timed-out run's strand live.
A reading that errors counts as not mid-turn, so an unreadable advisor never stretches the deferral beyond the up-line rule.
No decision reads the wait mark (`PATTERN-wait-mark-display-only`).

`Engine.Run` builds the notice sender and an `advisorWork` bound to it before the chair starts, `startSeat` sets the chair's spec `HoldDeferral` to the work's `outstanding` (an advisor's spec gets none), and each started advisor is added to the work as it starts.
`Engine.Probe` creates the work, passes its `outstanding` to the chair's probe through `probeSeat`, and keeps it on the returned `LiveTable` in a new unexported field;
`Engine.Resume` binds its sender to that work and adds every found advisor, so an attached chair defers as a fresh one does.
A `LiveTable` built without a probe carries no work, and `Resume` then gives the attached chair nothing new to read.
The settle interval and the maximum deferral are shuttle's (card 1): a chair held while every counted advisor is idle with its up line landed notifies once after the settle interval, and a chair held past the maximum deferral notifies once whatever its advisors do.

Bound: the deferral only delays the chair's notice and never past shuttle's maximum deferral, so an advisor wedged mid-turn hides a chair stall for at most that long; the table timeout cannot bound it, since every seat shares it and the chair starts first.

Tests:
- `seatfake_test.go`: `fakeHandle` answers `SessionState` from a scripted state the test can change.
- `engine_test.go`: a new table-driven `TestEngine_Run_ChairHoldDeferral` reads the predicate from the chair spec the fake shuttle recorded and calls it, with rows: an advisor mid-turn defers; an advisor whose up line has not landed defers; idle advisors with landed up lines do not; an advisor whose run ended (done, and timed out) never defers; a session-state error does not defer; no advisor spec carries a predicate; and the probe-then-resume path wires the attached chair's predicate to the found advisors.
- `smoke_seats_test.go` (`llm` tier, compiled by every gate and run by hand): the advisor stencil `testdata/smoke-advisor.md` changes to wait for the chair's question before it writes its output, and the test polls the advisor's session state through `shuttleengine.ReadSessionStates` while the table runs, asserting a reading whose history holds a busy turn after an idle stall: the turn the chair's SendMessage started reads mid-turn by shuttle's session-state fold.

Raise `internal/seatengine`'s row in `cmd/lyx/testdata/test-budget.yaml` by the new top-level test.
Describe the chair's hold deferral in `internal/seatengine/doc.go`'s fresh-run and resume sections, naming the Handle's new method.

**ImpactSummary:** Seat Handle gains one method that shuttle's Run already has; two unexported helpers re-signed within the package; only the chair's spec changes.
