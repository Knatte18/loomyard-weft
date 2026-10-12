# Card 9 — shuttle-api-continue

**Create:**
- `plan:internal/shuttleengine#apiErrorContinueLine` -> `const apiErrorContinueLine = "[lyx] Your last turn ended on an API error while the machine slept; it is awake again. Carry on with your task where you stopped."`
- `plan:internal/shuttleengine#Run.continueAPIErrorStall` -> `func (run *Run) continueAPIErrorStall(held *heldTurnEnd) bool`

**Edit:**
- `internal/shuttleengine#RunState`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`
- `internal/shuttleengine/doc.go`

**Intent:**
Give an agent whose turn ended on an API error across a detected sleep one automatic continue line, once per turn (D9), so the board's seen case, an API error across a night's suspend, does not wait for the operator.

In Wait's held-turn-end branch, before the notice decision, `continueAPIErrorStall(held)` decides and sends; when it returns true the held turn end gets no notice and no pending hold.
It sends only when all of these hold:
- the run is not covered (`run.covered`, card 8), so it never fires for a covered run;
- the run's session state (`run.SessionState()`) names the newest turn end an API error (`SessionCauseAPIError`);
- a sleep of at least `pauserecord.SleepTolerance` lies after `run.turnSince`, the run clock's time when Wait last read a turn start or began, from `run.sleeps` (card 8);
- `RunState.APIContinued` is false.
It then persists `APIContinued` true before typing, and types `apiErrorContinueLine` through `run.Send`, the guarded paste-submit (`PATTERN-pane-text-injection`).
A send that fails, the guard's `ErrSessionBusy` included, types nothing more, logs at Info, and returns false, so the stall is held and notified as today.
A held turn end that is not an API error sets `APIContinued` false again, so the flag spans exactly the turn the continue resumed, and a second API-error stall on that same turn is held and notified as today; persisting it keeps a reattached Wait from sending twice.
`Run` gains `turnSince`, set where `pollEventsTick` reports a turn start read (`run.turnStartRead`) and at Wait's start.

Bound: only on an API-error cause, only across a detected sleep, at most once per turn, only through the guarded paste-submit, and nothing when the guard refuses.

Test: `TestWait_HeldTurnEndNotice` gains rows over its session-signal engine, which gains an API-error turn-end line, and a clock that is an `AwakeClock` with a sleep jump in its script: an API-error stop after a detected sleep types one continue line and sends no notice; a second API-error stop after the continue's turn start is notified and types nothing; an API-error stop with no sleep is notified as today; the same stop with the run covered types nothing; a continue whose send fails busy is notified.
`doc.go` names the continue in the held-turn-end paragraph.

**ImpactSummary:** One new branch in Wait's held-turn-end path and one persisted RunState flag; behaviour changes only for an API-error stall across a detected sleep.
