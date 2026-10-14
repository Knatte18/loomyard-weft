# Card 4 — shuttle-conditioned-send

**Edit:**
- `internal/shuttleengine#sendContext`
- `internal/shuttleengine#sendVerified`
- `internal/shuttleengine#clearUnlandedText`
- `internal/shuttleengine#clearableBoxEngine.ParseSessionSignals`
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/shuttleengine#SendCondition` -> `type SendCondition struct`
- `plan:internal/shuttleengine#ErrSendConditionUnmet` -> `var ErrSendConditionUnmet error`
- `plan:internal/shuttleengine#Runner.SendConditioned` -> `func (r *Runner) SendConditioned(guid, text string, want SendCondition) (turnStarted bool, err error)`
- `plan:internal/shuttleengine#conditionedReading` -> `func conditionedReading(sc sendContext, want SendCondition) error`
- `plan:internal/shuttleengine#awaitTurnStart` -> `func awaitTurnStart(sc sendContext, sentAt int64) bool`
- `plan:internal/shuttleengine#typeVerified` -> `func typeVerified(sc sendContext, text string) error`
- `internal/shuttleengine/conditionedsend.go`
- `internal/shuttleengine/conditionedsend_test.go`

**Intent:**
Shuttle gains a conditioned send, which `revive` uses (card 10) to give one line to a session stalled on an API error (D5).
It lives in the new file `conditionedsend.go`, `package shuttleengine`.

`SendCondition` names the session state its ready reading must show: `State SessionStateName` and `Cause string`.
`ErrSendConditionUnmet` is `errors.New("shuttle: the session does not read the send's condition")`.

`Runner.SendConditioned(guid, text, want)` validates the text, finds the run as `Runner.Send` does and builds its send context with the new `sendContext` field `keepUnlanded` set.
It then takes one ready reading through `conditionedReading`; on a refusal it returns the refusal and types nothing.
It records the events file's size, types through `typeVerified`, and once the send lands waits through `awaitTurnStart`.
It returns `turnStarted` true when a turn start past that offset reads within the wait, and false with a nil error for a send that landed with none seen.

`conditionedReading` is one reading with no wait, no settle and no key: a fixed number of pane captures and event-file reads.
It refuses at once:
- with an error wrapping `ErrRewindPickerOpen` when the capture shows the rewind picker; no key goes, so the picker stays open.
- with an error wrapping `ErrSendConditionUnmet`, naming the state read, when the run's session state, read as `readRunSessionState` reads it, has a name or cause other than `want`'s.
  A busy session is refused here.
  So is an unmatched turn start: a turn start after the API-error turn end folds the state to busy, with no idle override and no `turnStartHold`, so the state check is the turn-start refusal and no separate hold branch is written.
- with a `*SessionBusyError` for any hold the ready reading names:
  a pane too short or showing no input box;
  a non-empty input box, a draft or a lone stray character alike, with `Draft` set;
  and operator activity in the pane, asked last.
It never clears a stray character and never closes the picker.

`typeVerified` is the typing half of `sendVerified`, cut out with no change but one: the paste path, or the typing, appearance and submit path, with their windows.
The `retryLanded` check stays out of the cut:
`sendVerified` becomes `awaitIdleSession`, then `retryLanded` on the paste path, then `typeVerified`, so every existing send behaves as today.
`Runner.SendConditioned` therefore never consults `pending-send.json`, and a line an earlier unlanded send left pending is pasted again instead of counted as landed.
`clearUnlandedText` skips its clear when `sc.keepUnlanded` is set, returning not landed with the note "the unlanded text was left in the input box".
A conditioned send that does not land therefore fails with `ErrSubmissionNotLanded` and leaves its text in the box.

`awaitTurnStart` polls `turnStartedSince` on the send context's clock from the landing for at most one submit window (`submitConfirmTimeout`), at `idlePollInitial` doubling to `idlePollMax`, with `windowPollCap` as its read bound.

Bound: compared with `Runner.Send`, a conditioned send adds refusals and drops the picker close, the stray-character clear and the unlanded-text clear, then adds the turn-start wait.
So it never types where today's send would not, and plays no key beyond its typed line and its submit.
It holds its caller for one ready reading, at most one submit window from the start of typing to the landing, and at most one more submit window for the turn start.
Each pane capture inside that span waits on reed's per-worktree operation lock and a tmux call with no timeout of their own, so the windows bound the polling, not a hung reed or tmux call.

`doc.go` gains a "Conditioned send" paragraph stating the at-once refusals, the dropped clears and the turn-start wait.

`TestRunner_SendConditioned` in `conditionedsend_test.go` is the one new top-level test, for a surface no existing test reaches.
It runs a `Runner` over a `tuiReed` and a `pickerEngine`, which parses the session-fake signals, reads the box after the caret and reads the picker.
The run's events file holds `START` then `APIERR`, so the session reads idle-stalled with cause `api-error`; `want` is that state.
The clock is a `hookClock` (`gate_test.go`), whose sleep hook the clean row uses to append a line to the events file once the box reads submitted.
Rows:
- clean: the line lands, a `START` appended by the clock's sleep hook after the landing reads as a turn start, and the result is `turnStarted` true.
- landed with no turn start: no `START` follows within the submit window; `turnStarted` false and a nil error.
- busy: the events file ends with `START`, so the state reads busy; `ErrSendConditionUnmet`, no text typed and no key sent.
- unmatched turn start after the stall: `APIERR` then `START` with the pane idle; `ErrSendConditionUnmet` at once, nothing typed, and no wait on the clock.
- draft: the box holds `half a thought`; a `*SessionBusyError` with `Draft` true, no key sent and the box unchanged.
- lone stray character: the box holds `x`; the same refusal, and no clear key sent.
- operator active: `ActiveQueue` reads active; a `*SessionBusyError`, nothing typed.
- no input box: the pane shows no input box; a `*SessionBusyError`, nothing typed, and the clock where it was.
- rewind picker: the capture shows `REWIND-PICKER`; an error wrapping `ErrRewindPickerOpen`, with no key sent.
- not landed: `swallowEnterUntil` past the window; `ErrSubmissionNotLanded`, and the box still holds the sent text, with no clear key sent.
- pending record from an earlier send, over a `pasteEngine`: `pending-send.json` holds the same text with a `START` past its offset, then the events file reads `START` and `APIERR`; the line goes out through `PasteSubmit`, not typed, and the result is `turnStarted` as scripted.
  The paste path is the one production's engine takes, so this row also pins that it is taken.

`clearableBoxEngine.ParseSessionSignals` (`run_test.go`) gains the case `trimmed == "APIERR"`, mapped to a signal of kind `SessionSignalAPIErrorTurnEnd`.
A `pasteEngine` embeds `clearableBoxEngine` through `tuiEngine`, so without the case it drops `APIERR`, reads the events file as an unmatched turn start and refuses the pending-record row with `ErrSendConditionUnmet` before pasting.
Each refusal row asserts the fake clock advanced zero.

`test-budget.yaml`'s `internal/shuttleengine` row rises from 171 to 172.

**ImpactSummary:** `sendVerified` is split with no behavior change for its callers, and `clearUnlandedText` gains one field-guarded skip that only the new conditioned send sets.

**Commit:** 4: shuttle: a conditioned send that refuses at once and waits for the turn start
