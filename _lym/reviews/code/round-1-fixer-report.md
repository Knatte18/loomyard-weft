---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
---

# Fixer report, round 1

Every finding is fixed as proposed.
The gate (`lym gate test ./internal/shuttleengine ./internal/shuttleengine/claudeengine`) passes.
Two commits hold the work: the review's findings, then the self-check's follow-ups.
The review's findings share `hold.go`, `run.go` and the test files, and the round rule makes all edits in one round, so they are one commit instead of one per finding.

## What changed

- F1: `Run.promptSendOffset` holds the events file's size read as the prompt line goes out, beside `promptSentAt`, under `sendMu`.
  `retypeLostPrompt` judges `turnStartedSince` from it, and `PromptOffset` and `promptTurnStartedSince` are unchanged.
  The offset is read where the text is pasted or typed (`sendContext.sentOffset`, set in `pasteVerified` and `typeVerified`), not before the idle wait.
  New rows: an operator turn seeded before the attach, and one appended while the attach's send waits.
- F2: `retypeLostPrompt` latches `lostPromptRetyped` only after `typePendingPrompt` returns, and leaves it unset for a `*SessionBusyError`.
  The doc comment, `doc.go` and the landing summary say a retype held busy does not count.
  New row: the first retype is held busy by an active operator, and the next tick types the line, two typings in all.
- F3: The idle layout of the board entry (`ok` reply, a done timing line, top rule, empty caret line, bottom rule, bypass-permissions footer) reads an empty box.
  This holds plain and through `PlainBoxCapture` of a styled form with a non-breaking space after the caret.
  Both are rows of `TestInputBoxText`, and no reader code changed.
  `_lym/landing/summary.md` states the outcome.
- F4: `TestStart_RetypesAPromptLineThatStartedNoTurn` runs `Runner.Run` over a fake that records no turn start (typed twice) and one that records it (typed once).
- F5: `TestStartup_SendFailuresTearDown` gains rows for an always-busy prompt (`startPromptBusyRetries+1` idle waits, nothing sent) and a timeout inside the first window (one wait).
  The foreign-box row asserts exactly one paste.
  The wait count comes from the idle readings: a reading at the same fake-clock instant as the one before opens a new wait.
- F6: `sendStartPrompt` builds each send context with `deadline = run.deadline`.
  `sendThroughPause` keeps the context's deadline across a pause.
  `retypeLostPrompt` returns without typing when the run clock is not before `run.deadline`, and passes the deadline to the retype's send.
  Rows: the always-busy and the deadline rows above, and a Wait row whose timeout equals the submit window.
- F7: Rows in `TestWait_RetypesAPromptLineThatStartedNoTurn` for a pause covering the run after the attach typing, an engine with no `SessionSignalParser`, and a timeout shorter than the submit window.
  Each types the line once.
- F8: Rows in `TestStartup_SendFailuresTearDown` for a `StyledBoxReader` engine (the saved capture is the styled pane, not the plain one) and for a failing failure capture (the saved capture is the boot capture).
- F9: `abandonStartup`'s doc says it saves `lastStartupCapture`: the startup probe's last capture, or the pane at a failed prompt delivery.
- F10: The `promptDelivered` comment says no later `DeliverPendingPrompt` call retypes the prompt.

Gate repair: a comment-style gate found wrapped or over-long comment lines the round added, in `attach_test.go`, `doc.go`, `hold.go`, `run.go` and `startup_test.go`.
Each was rewritten as one sentence per line under 120 characters, and no code changed.
A second gate pass found one more wrapped doc line, on `TestStart_PromptSendWaitsOutAPauseLongerThanTheRunTimeout`, and it was rewritten the same way.

Mutation checks: with the retype's deadline guard, the sent-offset assignment, or the pause deadline shift removed, the row built for each fails.

## Self-check

The four lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.
No fork returned without a report.

- fix-code-correctness, LOW, no test fails without the retype's deadline guard: fixed. Row "a retype due at the run's deadline is not typed" (timeout equal to the submit window); the guard's removal fails it.
- fix-code-correctness, LOW, the held-busy retype logs at Warn on every tick for an engine with no idle wait: fixed. It logs at Info.
- fix-consistency, LOW, the landing summary still described the pre-fix retype and tests: fixed. `_lym/landing/summary.md` carries the new conditions, the busy rule, the deadline and the new tests.
- fix-affected-set, LOW, the retype's send was not cut to the run deadline: fixed. `typePendingPrompt` takes a deadline; the retype passes `run.deadline` and `DeliverPendingPrompt` passes none.
- fix-proposal, MEDIUM, the carried send deadline fails a start whose pause outlasts the timeout: fixed. `sendThroughPause` moves the carried deadline later by the time it waited out a pause.
  New test `TestStart_PromptSendWaitsOutAPauseLongerThanTheRunTimeout`, which fails without the shift.
- fix-proposal, LOW, the offset was read before the send's idle waits, so an operator turn during them cancelled the retype: fixed differently.
  The proposed change, read the offset just before `sendThroughPause`, was made first and does not cover a wait before the typing.
  The send now reports the offset where it types, and a row pins it.
- fix-proposal, LOW, an unlatched busy retry holds `Wait` for a full idle window on each tick: disputed, every fix worse.

## Disputed

- fix-proposal LOW, the busy retype's per-tick idle wait.
  Ground: every fix worse.
  Any cap on busy retypes leaves a dropped line untyped after the cap, which is F2's premise again, and a shorter wait cannot land a send the idle reading holds.
  The wait is cut to the run deadline since F6, and `deliverPromptAtHold` and `deliverPromptOnAttach` already wait inside the tick the same way.
  Evidence: `internal/shuttleengine/hold.go:189`, `internal/shuttleengine/hold.go:142`, `internal/shuttleengine/hold.go:162`.

## Deferred

None.

## Rulings

None reached this round.
