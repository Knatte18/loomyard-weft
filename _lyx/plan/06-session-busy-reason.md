# Card 6 — a held send carries its readiness reason apart from the pane tail

**Create:**
- `internal/shuttleengine#SessionBusyError`
- `internal/shuttleengine#SessionBusyError.Error`
- `internal/shuttleengine#SessionBusyError.Unwrap`

**Edit:**
- `internal/shuttleengine#awaitIdleSession`
- `internal/shuttleengine#TestSend_WaitsForIdleSession`

**Intent:**
D6 needs the resume to report a refused send by the readiness reason the guard named, without the pane tail.
Today that reason is only inside the formatted `ErrSessionBusy` message, before "The pane's last lines".
This card exposes it as data; card 8 reads it.

- In `send.go`, add the exported `SessionBusyError` with two exported fields, `Reason string` and `Err error`.
  `Reason` is the hold the idle wait's last reading named, such as "a turn is running", "no input box is on screen; a menu or picker may be open", the draft or the operator-activity hold, with no pane tail.
  `Err` is the full error, which wraps `ErrSessionBusy`.
  `Error` returns `Err.Error()` and `Unwrap` returns `Err`.
  The message every caller and test sees is therefore unchanged, and `errors.Is(err, ErrSessionBusy)` still holds.
  The fields are exported so a caller's fake, as in card 8's resume test, can build one.
- `awaitIdleSession` returns `&SessionBusyError{Reason: hold, Err: <today's fmt.Errorf>}` at its operator-activity hold for the engine without `SessionCycler`.
  It returns `&SessionBusyError{Reason: busy, Err: <today's fmt.Errorf>}` when the idle wait runs out.
  Both messages stay byte for byte as they are.
- The other `ErrSessionBusy` errors (`ErrSentTextInDraft` and the box-holds-other-text failure in `confirmPasted`) are failures after typing began, not idle-wait holds, and stay plain wrapped errors.
  Card 8's resume reports those by their message.
- `TestSend_WaitsForIdleSession`: in the `wantBusy` branch, assert `errors.As(err, &busy)` with `busy *SessionBusyError`, and that `busy.Reason` is non-empty and does not contain "The pane's last lines".
  Add a `wantReason string` row field, asserted with `strings.HasPrefix(busy.Reason, tt.wantReason)` when set, and set it on three rows:
  - "unmatched turn start against a busy pane fails busy with the reading's reason": `a turn is running`.
  - "draft in the box fails busy with the reading's reason": `the input box holds a draft`.
  - "an operator active through the whole window fails busy naming the activity": `the operator typed in this pane`.

  A `Reason` that held the whole message, or the tail, fails the first assertion, and a wrong reason fails the prefix.

**ImpactSummary:** Two return sites in `awaitIdleSession` change type, not text; every `errors.Is(..., ErrSessionBusy)` caller (wait loop, orch watcher, loom start and resume, seat notices, pause resume) keeps matching.

**Commit:** 6: Carry a held send's readiness reason in SessionBusyError
