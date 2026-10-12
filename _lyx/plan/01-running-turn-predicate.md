# Card 1 — running-turn predicate on the session state

**Create:**
- `plan:internal/shuttleengine#SessionState.RunningTurn` -> `func (s SessionState) RunningTurn() bool`

**Edit:**
- `internal/shuttleengine#judgeReadiness`
- `internal/shuttleengine#TestSessionFold`

**Intent:**
D1: `shuttleengine` gains one provider-neutral predicate for a running turn, shared by the pause and by send readiness.

- In `internal/shuttleengine/sessionstate.go`, add the method `SessionState.RunningTurn`: true exactly when `Name` is `SessionBusy` and `Cause` is anything but `SessionCauseBackground`.
  Its godoc says a busy state whose cause is background is a turn that ended with background work still running, so it is not a running turn.
- `judgeReadiness` in `readiness.go` replaces its inline `state.Cause != SessionCauseBackground` test in the `SessionBusy` case with `state.RunningTurn()`, keeping the reason "a turn is running" and every other branch as it is.
  The two callers still apply the rule to different states: `sessionReadiness` first turns a released turn start into idle-stalled, and the pause reads the raw folded state (card 8).
- `TestSessionFold`: add a `wantRunningTurn bool` field to the row struct and assert `fold.State().RunningTurn() == tt.wantRunningTurn` beside the existing `State()` check.
  Set it true on every row whose `wantState` is `SessionBusy` with a cause other than background (the turn-start rows), and leave it false on the rest, the row "a turn end with outstanding tasks is busy on the background" included, which is the row that fails if the predicate reads background as running.

**ImpactSummary:** One new method with no caller outside `shuttleengine` until card 8; `judgeReadiness` keeps its outputs, so every existing readiness and send test holds.

**Commit:** 1: Add SessionState.RunningTurn, shared by send readiness
