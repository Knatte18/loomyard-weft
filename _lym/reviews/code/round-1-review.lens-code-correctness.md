```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I traced each function and branch the diff adds or changes through concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

- **`sendStartPrompt` (run.go:892)**
  - A first send held busy, then idle at retry 1: it sends again and returns nil. Correct.
  - Busy on all three sends: it returns the third `*SessionBusyError`, after `startPromptBusyRetries` = 2 retries. Correct.
  - `run.deadline == now` after the first busy hold: `!Before` is true, so it returns the busy error without retrying. Correct.
  - A zero-Timeout spec, where the deadline equals createdAt: no retry. Correct.
  - Can a retry type the line twice? Every `&amp;SessionBusyError` site is before the paste: send.go:198 and send.go:228 in the idle wait, and send.go:648 and send.go:651 in `clearBoxBeforeCommand`, which only a `startsNoTurn` send reaches. The prompt send never reaches it. `ErrSentTextInDraft` and `ErrForeignBoxText` are plain sentinels, not `*SessionBusyError`. So no retried error follows a paste. Correct.
  - Does a retry honour the deadline? A retry that starts 1 s before `run.deadline` runs a full idle wait plus the submit window past it. `run.newSendContext()` leaves `sc.deadline` zero (send.go:92–106), so `awaitIdleSession`'s limit (send.go:209–212) is never cut to the run deadline. Finding F1.
  - A `waitWhileCovered` failure inside `sendThroughPause`: it is not a `*SessionBusyError`, so it returns. Correct.
- **`loadSkillsThenPrompt` failure branch (run.go:870–876)**
  - `ErrSubmissionNotLanded` with a late turn start: returns the run, leaves `promptSentAt` zero, and the retype stays inert. Correct, because that turn started.
  - Any other error: `keepFailureCapture` runs, then `abandonStartup` saves `lastStartupCapture`. The cause is not nil, so no excerpt is appended (wait.go:864). Correct.
- **Success branch (run.go:877–879)**: `promptSentAt` is set under `sendMu`. Wait has not started yet, so nothing races it. Correct.
- **`keepFailureCapture` (run.go:906)**
  - A `StyledBoxReader` engine: saves the styled capture.
  - A plain engine: saves the plain capture.
  - A capture error, for example a dead pane: keeps the boot capture.
  - No production code parses `startup-capture.txt`: the grep found only the writer. Correct.
- **`DeliverPendingPrompt` / `typePendingPrompt` split (run.go:946–994)**
  - sent=true: `promptDelivered` is set, as before.
  - sent=false with nil, and an error: it stays unset, as before.
  - `promptSentAt` is set only after a send succeeds.
  - Offset persist fails: returns the error and leaves `promptSentAt` unchanged.
  - Both callers of `typePendingPrompt` hold `sendMu`. Correct.
- **`retypeLostPrompt` (hold.go:189)**
  - Covered: returns. An engine that parses no signals: returns. Zero `promptSentAt`: returns.
  - Elapsed time below the submit window: returns. An attach typing on the same tick sets `promptSentAt` = now, so it never retypes on that tick. Correct.
  - A turn start past the offset: it clears `promptSentAt`, and later ticks are cheap. Correct.
  - No turn start: it latches `lostPromptRetyped` and types through `typePendingPrompt`, which re-checks `promptTurnStartedSince`. Correct.
  - Lock order is `sendMu` then `recordMu`, the same as the existing `DeliverPendingPrompt`. Correct.
  - A retype held busy, which typed nothing: the latch is already set, so the line is never tried again. Finding F2.
  - A pause landing during the retype's idle wait: `sendThroughPause` waits out the whole pause inside the `Wait` tick, with `sendMu` held. Finding F3.
- **`Wait` tick wiring (wait.go:313)**: it is called after `deliverPromptOnAttach` and before `pollEventsTick`. Correct.

**Findings**

- **F1 — LOW, design.** Location: `internal/shuttleengine/run.go:892` (`sendStartPrompt`).
  - Summary: the retry's deadline check runs only before each resend. The resend's idle wait and submit window are not cut to `run.deadline`, so the start can run past the deadline by up to the `send_ready_timeout_s` window (60 s by default) plus `submit_confirm_timeout_s` (30 s by default).
  - Evidence: `run.go:894` uses `run.newSendContext()`, whose deadline is zero (`send.go:92–106`). `awaitIdleSession` cuts the wait only when `sc.deadline` is non-zero (`send.go:209–212`).
- **F2 — MEDIUM, design.** Location: `internal/shuttleengine/hold.go:205–212` (`retypeLostPrompt`).
  - Summary: `lostPromptRetyped` is latched before the send. A retype held by the idle wait typed nothing, yet it still uses up the handle's one retype. This includes the "no input box" misread that issue #137 reports. The lost line is then never typed, and the run sits idle until its timeout.
  - Evidence: `hold.go:205` sets the latch before `typePendingPrompt`. The `err != nil` case only logs (`hold.go:208–209`). All `*SessionBusyError` sites are pre-paste (`send.go:198`, `send.go:228`), so such an attempt typed nothing.
- **F3 — LOW, design.** Location: `internal/shuttleengine/hold.go:206`.
  - Summary: a pause that starts during the retype's idle wait makes `sendThroughPause` call `waitWhileCovered` inside `Wait`'s tick, with `sendMu` held. Wait's liveness and deadline checks stall for the whole pause. The `run.covered` guard at `hold.go:190` covers only a pause that was read before the call.
  - Evidence: `run.go:921–932` (`sendThroughPause` loops on `waitWhileCovered`); `wait.go:311–313`.

Tool calls:

1. Read `send.go` lines 619–708, to check whether any `*SessionBusyError` site (send.go:648 and send.go:651) comes after a paste. The manifest held the grep hits only, not their surrounding code.
2. Grepped for readers of `startupCaptureFileName` and `startup-capture.txt`, to check whether a styled capture breaks any parser. The manifest held no reader search.
