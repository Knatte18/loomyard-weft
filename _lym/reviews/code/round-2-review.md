---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: typeVerified and pasteVerified write the events size read just before the text goes out into sc.sentOffset; loadSkillsThenPrompt and typePendingPrompt keep it as promptSendOffset, and retypeLostPrompt judges turnStartedSince from it. The operatorTurn and operatorTurnDuringSend rows of TestWait_RetypesAPromptLineThatStartedNoTurn pin it.
  - id: F2
    status: resolved
    answer: retypeLostPrompt sets lostPromptRetyped only after typePendingPrompt returns with no *SessionBusyError; the holdFirstRetype row pins two typings.
  - id: F3
    status: resolved
    answer: TestInputBoxText gains the plain and styled rows of the board entry's idle layout (the styled row carries the non-breaking space after the caret), both reading an empty box, and summary.md states that outcome and that the cause stays unknown.
  - id: F4
    status: resolved
    answer: TestStart_RetypesAPromptLineThatStartedNoTurn drives Runner.Run with and without a turn start for the prompt and asserts two and one typings.
  - id: F5
    status: resolved
    answer: TestStartup_SendFailuresTearDown gains the retry-cap row (startPromptBusyRetries+1 waits, nothing sent), the deadline row (one wait, nothing sent) and a pastes count of one on the foreign-box row.
  - id: F6
    status: resolved
    answer: sendStartPrompt sets sc.deadline to run.deadline per send, sendThroughPause keeps the deadline (moved by the pause wait) across rebuilt contexts, typePendingPrompt passes the retype's deadline into the send, and retypeLostPrompt returns once the deadline has passed; the F5 deadline row covers it as the fix asked.
  - id: F7
    status: resolved
    answer: The pauseCoversRun, noSignals and timeout window/2 rows each assert one typing.
  - id: F8
    status: resolved
    answer: The styled-pane row asserts the saved capture holds the styled screen and lacks the plain one; the failing-capture row asserts the boot screen is kept.
  - id: F9
    status: resolved
    answer: abandonStartup's doc now names lastStartupCapture as the probe's last capture or the pane at a failed prompt delivery.
  - id: F10
    status: resolved
    answer: The promptDelivered comment now says no later DeliverPendingPrompt call retypes it.
---

Every prior finding is resolved by the fix round, and this incremental round turns up no BLOCKING defect on a changed hunk.
`lym gate test ./internal/shuttleengine ./internal/shuttleengine/claudeengine` passes at tier0 on HEAD (2265 passed, 0 failed).

The focus questions each settle as not a defect:

- The retype watch starts at the events size read as the watched text goes out, through `sendContext.sentOffset`, so an operator turn from before the attach typing does not cancel it.
- The one-retype latch is set only after a send that was not held with a `*SessionBusyError`.
- The board entry's idle layout reads as an empty box in `InputBoxText`, plain and styled, and the landing summary states the outcome.
- Start-then-Wait and attach-then-Wait tests assert two typings of a dropped line and one when the typing records a turn start.
- The busy retry's cap, deadline stop, `*SessionBusyError` limit and single foreign-box paste are pinned.
  Only `awaitIdleSession` and `clearBoxBeforeCommand` build a `*SessionBusyError` in the send path, both before anything is typed, and the prompt is never a no-turn send, so the retry cannot type twice.
- The start sends and the retype carry the run deadline into the idle wait and submit window.
- Each retype guard and each `keepFailureCapture` branch has a row that fails without it.
- The `abandonStartup` doc and the `promptDelivered` comment match the code.

## Rejected

- Assistant's carried F6 (an Enter in `confirmPasted` can follow the deadline): rejected.
  `confirmPasted` is unchanged by this diff, the paste itself goes out only inside the deadline-cut window, and the late Enter re-submits that same text at most one `SubmitSettle` past `closeAt`.
  The fix applied the prior F6's prescribed change and test row in full.
  The assistant's coverage note, that the deadline row does not pin the deadline's cut of the first idle wait, is a new LOW coverage gap, which this incremental round may not raise.

## Notes below the round's bar

These are not findings, since an incremental round raises only BLOCKING findings; they are recorded for the parent.

- `retypeLostPrompt` retries a busy retype on every Wait tick, and each try runs a full idle wait of up to `send_ready_timeout_s` (cut to the run deadline) on Wait's goroutine while holding `sendMu`.
  A session that shows no input box for a long stretch, the #137 symptom, would then stall event polling, hold notices and liveness checks for up to that window per tick until the deadline.
- `sendStartPrompt` decides a resend, and sets the next send's deadline, from the unmoved `run.deadline`.
  After a pause waited out inside one send, a following busy hold therefore gets no resend, or a resend whose idle wait ends at once, though the summary says the pause moves the deadline later.
