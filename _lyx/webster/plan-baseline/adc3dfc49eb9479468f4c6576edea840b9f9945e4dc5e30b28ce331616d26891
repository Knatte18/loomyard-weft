# Card 2 — hold-deferral-held-advisor

**Edit:**
- `internal/seatengine#advisorWork.outstanding`
- `internal/seatengine#TestEngine_Run_ChairHoldDeferral`
- `internal/seatengine/doc.go`

**Intent:**
D2: a held or dead advisor never defers the chair's held-turn notice.

`advisorWork.outstanding` reads each advisor's session state before it looks at the advisor's up line.
For an advisor whose run has not ended, it calls `run.handle.SessionState()` first; when that reading succeeds with a name of `shuttleengine.SessionAsking`, `SessionIdleStalled`, `SessionIdleDone` or `SessionDead`, the advisor is skipped.
Otherwise the advisor counts when its up line has not landed (`sender.wasSent` false) or the reading is busy with cause turn, as today.
A reading that errors counts as neither held nor busy, so only the up-line clause can still count that advisor.
The session reading alone decides held; the held-turn wait marker is not read.
`holdNoticeMaxDeferral` and shuttle's settle interval are unchanged, so the 10-minute cap stays the backstop for an advisor wedged mid-turn.
The method's doc comment states the new order and the four held-or-dead readings.

What this lets through: a chair whose advisor is stopped on a question, a permission prompt or a stalled or done turn, or whose advisor's process is gone, now notifies the parent after the settle interval, before that advisor's up line lands; the deferral still holds for a live advisor that is not held and whose up line has not landed, and for an advisor mid-turn.

`TestEngine_Run_ChairHoldDeferral`:

- The `act` field's comment becomes "scripts the advisor once it is counted, after its up line landed unless upLineStuck".
- The row "an advisor whose up line has not landed defers" gains `act: func(a *fakeHandle) { a.setSession(shuttleengine.RunSessionState{}, nil) }`, so a live, not-held, not-busy advisor is counted by the up-line clause alone; today the fixture leaves that advisor mid-turn, so the row cannot tell the two clauses apart.
- New rows, each with `upLineStuck: true` and `wantDefers` false, whose `act` sets the advisor's session to a `RunSessionState` named `asking`, `idle-stalled`, `idle-done` and `dead` in turn: an advisor so held or dead whose up line has not landed does not defer.
  With the old order each of these rows defers on the unlanded up line, so each fails on the old code.
- A new row with a landed up line and `wantDefers` false whose `act` sets the session to `dead`: a dead advisor never counts once its up line landed either.
- A new row with `upLineStuck: true` and `wantDefers` true whose `act` calls `a.setSession(midTurn, errors.New("unreadable"))`: an unreadable session with its up line not landed still defers through the up-line clause.

The existing rows for an advisor mid-turn, idle advisors with landed up lines, an ended or timed-out advisor and an attached chair stay as they are.

`internal/seatengine/doc.go`'s paragraph on the hold deferral states that an advisor whose session reads `asking`, `idle-stalled`, `idle-done` or `dead` never counts, whether or not its up line has landed.

**ImpactSummary:** One unexported predicate behind the chair's `HoldDeferral`; its only test is `TestEngine_Run_ChairHoldDeferral`, extended here, and no signature changes.

**Commit:** 2: seat: a held or dead advisor never defers the chair's held-turn notice
