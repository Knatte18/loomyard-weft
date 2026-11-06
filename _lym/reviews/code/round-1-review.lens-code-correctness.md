```yaml
lens: code-correctness
outcome: reported
```

**Cases run**
- `loadSkillTurn`, first turn, no API error, turn ends: returns the turn end; correct.
- `loadSkillTurn`, timeout with a turn start read, no end: waits until the 15-minute open bound, then returns `ended=false`. Correct against its own doc, but see F1.
- `loadSkillTurn`, timeout with no turn start read (engine without `SessionSignalParser`, or the hook line missing): returns `ended=false` at `timeout`. **Not correct**, see F1.
- `loadSkillTurn`, API error pending: the `else if` timeout branch is skipped, so the hold is unbounded. The back-off runs 30s, 60s, 120s, then the 5-minute ceiling. A model-rejected error returns `*loadModelRejectedError`. Correct.
- Retype with `ErrSessionBusy`: sets `retypeAt` to now plus `backoff` and continues. The backoff stays doubled, not reset. Correct.
- Retype with another send error: returns the wrapped error, which `abandonLoad` turns into `OutcomeDied`. Correct.
- Retype with a dead pane: `reed.Status` is checked before the send, so `died` is returned. Correct.
- `apiErrorRead`: three errors give waits of 30s, 60s and 120s, which matches the test's `7*apiErrorRetryBackoff`. Correct.
- `turnStartRead` while pending (operator or revive): clears `pending` and resets `began`. Correct.
- `readLoadTurn`, partial or unparseable data: reads nothing and leaves the offset. Correct.
- `readLoadTurn` offset: advances past the whole batch, as the old code did. Events read during a hold are not re-read. Correct.
- `ParseSessionSignals` `consumed` is dropped: a trailing stamp line with no payload yet loses only its time, since the later payload line still carries `hook_event_name`. No signal is lost. Correct.
- `noticeLoadHold`: sends once at the second error, skips interactive runs and a nil notifier, and logs a notifier error. Correct.
- `endLoadHold`: shifts the deadline by now minus `since` on every return path, including the model-rejected and send-error returns. Correct, apart from F2.
- Send deadline during the hold: `newSendContext` sets no deadline, since only `sendWithin` sets one. A hold longer than `Spec.Timeout` therefore does not break the retype send. Correct.
- `abandonLoad` with a rejected model: finalizes `OutcomeAPIError`, sets `Result.APIError` and removes the strand. `startRetryable` is false because the outcome is not died. Correct, apart from F3.

**Findings**
- **F1 — MEDIUM, design — `internal/shuttleengine/loadturn.go:129-131` (`loadturn.go:282` in the diff).**
  - A timeout with no API-error signal read still returns `ended=false`. `settleLoadTurn` (`run.go:925-929`) then logs "skill skipped … timeout" and `loadSkillsThenPrompt` types the prompt.
  - Two paths still reach this: no turn start was read at all (an engine without `SessionSignalParser`, or a hook line not yet written), and an open turn still running at `loadTurnOpenBound` (15 minutes).
  - A network drop longer than 15 minutes, with the provider still retrying inside the turn, therefore still types the prompt into a failing API. That is the incident, shifted by 13 minutes.
  - The Goal says "the prompt line is not typed into a session whose API is failing". Nothing in the Goal licenses a 15-minute cap.
- **F2 — LOW, design — `internal/shuttleengine/loadturn.go:204-211`.**
  - `endLoadHold` extends the deadline by `now - since`. That span runs from the first API error to the end of the final successful load turn, so the successful turn's own run time is also added to the deadline.
  - It over-extends by up to one load turn.
- **F3 — LOW, design — `internal/shuttleengine/loadturn.go:217-225` with `run.go:824-826`.**
  - When every output file exists, `abandonLoad` returns `Result{}, nil` and `loadSkillsThenPrompt` returns at once.
  - The prompt offset is not persisted and the prompt is not typed, so `start` returns a run whose agent was never prompted.
  - `Spec.validate` refuses pre-existing outputs, so this is near-unreachable.
- **F4 — NIT, design — `internal/shuttleengine/loadturn.go:151-183`.**
  - The signal loop runs before the event loop, so an event-loop `apiErrorRead` (transcript marker) can set `pending` after a later turn-start signal in the same batch has cleared it.
  - If an engine reports a failure as both a `StopFailure` signal and a marked `Stop` event, `errors` and the back-off are double counted.
  - I did not confirm that Claude does this, so it is speculative.

**Tool calls**
1. Read `claudeengine/signals.go:55-120`: to check whether `consumed` from `ParseSessionSignals` matters, which my context lacked.
2. Read `send.go:895-935`: to check whether `newSendContext` carries the run deadline.
3. grep `sc.deadline` and `run.offset` in `send.go` and `wait.go`: to confirm only `sendWithin` sets `sc.deadline`.
