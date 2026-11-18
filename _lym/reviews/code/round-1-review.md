---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: assist:assist
    location: internal/shuttleengine/hold.go:201
    summary: retypeLostPrompt judges the watched send by any turn start past the durable PromptOffset, so an operator turn from before an attach delivery cancels the retype of a dropped line
    fix: >-
      Keep, beside promptSentAt and under sendMu, the events file's size read just before the watched typing
      (in loadSkillsThenPrompt that is sentAt; in typePendingPrompt read eventsSize(record.EventsPath) right before sendThroughPause).
      In retypeLostPrompt pass that offset to turnStartedSince in place of run.state.PromptOffset, and leave PromptOffset and promptTurnStartedSince unchanged.
      Add a row to TestWait_RetypesAPromptLineThatStartedNoTurn whose seeded events hold an operator turn start past the prompt offset before the attach,
      with no turn start after the attach typing, and assert the line is typed twice.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/hold.go:205
    summary: lostPromptRetyped is latched before the send, so a retype the idle wait held busy, which typed nothing, uses up the handle's one retype and the lost line is never typed
    fix: >-
      In retypeLostPrompt set lostPromptRetyped only after typePendingPrompt returns,
      and leave it unset when the error is a *SessionBusyError (errors.As), so a later tick tries again.
      Update the doc comment's at-most-once sentence to say a retype held busy before it typed does not count.
      Add a row to TestWait_RetypesAPromptLineThatStartedNoTurn whose first retype is held busy and whose next one lands, asserting two typings of the line.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/shuttleengine/run.go#sendStartPrompt
    summary: The board entry's first part asks to check the no-input-box reading against the idle layout it describes; the diff neither examines nor changes that reading, so a layout the reader misreads still fails the start after the retries
    fix: >-
      Build a Claude capture of the layout the board entry describes:
      the ok reply, a done timing line, the box's top rule, an empty caret line, the bottom rule and the bypass-permissions footer.
      Run it through Claude's InputBoxText in internal/shuttleengine/claudeengine, both plain and through PlainBoxCapture of a styled form, as a row of the existing InputBoxText table test.
      If it reads no box, fix inputBoxInterior or PlainBoxCapture so it reads an empty box.
      If it reads one, keep the row as the pin.
      Either way, state the outcome in _lym/landing/summary.md.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/run.go:877
    summary: The fresh-start retype has no test; deleting the promptSentAt assignment after a start's prompt send fails no test, though it is the main path of the second board part
    fix: >-
      Add a tier0 test, or a row in an existing Start-then-Wait test, that starts a run through Runner.Start with a fake that records no turn start for the prompt.
      Wait past one submit window and assert the prompt line is typed exactly twice.
      Add a sibling row where the typing records a turn start, and assert it is typed once.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/run.go:896
    summary: The busy retry's cap, its deadline stop and its limit to *SessionBusyError have no test; the always-busy and foreign-box rows assert no send or paste count
    fix: >-
      In TestStartup_SendFailuresTearDown, add a row whose idle reading never turns idle and assert the start tears down with ErrSessionBusy
      after exactly startPromptBusyRetries+1 idle waits, read from the fake clock's elapsed time or the reed's capture count, and no prompt typed.
      Add a row whose spec Timeout ends inside the first send window and assert one wait and no resend.
      Give the foreign-box row a count of pastes of the prompt and assert exactly one.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/run.go:894
    summary: The start retry checks the run deadline only before each resend, and neither its sends nor the Wait retype cut their idle wait or submit window to the run deadline, so either can type after the deadline
    fix: >-
      In sendStartPrompt build each send's context with sc.deadline = run.deadline, and have sendThroughPause keep the passed context's deadline when it rebuilds the context after a pause.
      In retypeLostPrompt return without typing when run.clock.Now() is not before run.deadline.
      Cover the deadline row through the F5 test row.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/hold.go:190
    summary: The retype's pause guard, its session-signal-parser guard and its submit-window wait have no test; removing any one leaves TestWait_RetypesAPromptLineThatStartedNoTurn green
    fix: >-
      Add rows to TestWait_RetypesAPromptLineThatStartedNoTurn.
      In one, a pause covers the run after the attach typing: assert one typing.
      In another, the engine implements no SessionSignalParser: assert one typing.
      In a third, the timeout is shorter than one submit window: assert one typing.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/run.go:906
    summary: keepFailureCapture's styled-capture branch for a StyledBoxReader engine and its keep-the-boot-capture fallback on a capture error have no test
    fix: >-
      In TestStartup_SendFailuresTearDown add a row with an engine that implements StyledBoxReader and a reed whose styled and plain captures differ,
      and assert the saved startup capture equals the styled capture.
      Add a row whose capture fails after the prompt send, and assert the saved file holds the boot capture.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/wait.go:805
    summary: abandonStartup's doc still says it saves the capture checkLivenessTick recorded, while a failed prompt delivery now saves the capture keepFailureCapture took
    fix: >-
      Reword the sentence at wait.go:805 to say it saves lastStartupCapture to startupCaptureFileName:
      the startup probe's last capture, or the pane at a failed prompt delivery.
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/run.go:326
    summary: The promptDelivered field comment says no later call retypes the prompt, but retypeLostPrompt now types it again after a delivery that set the latch
    fix: >-
      Reword the comment to say no later DeliverPendingPrompt call retypes the prompt.
---

### [MEDIUM] The retype watch reads turn starts from before the watched send

**Location:** internal/shuttleengine/hold.go:201

**Issue:** `retypeLostPrompt` calls `turnStartedSince` from `run.state.PromptOffset`.
On an attach, `typePendingPrompt` keeps a non-zero persisted offset.
An operator turn recorded past that offset before the restart leaves the prompt pending, by `DeliverPendingPrompt`'s own contract (run.go:938).
If the attach's typing is then dropped, the watch finds the older operator turn, clears `promptSentAt` and never retypes.
That is the "also after a restart" case the board entry names.

### [MEDIUM] A busy retype uses up the one retype

**Location:** internal/shuttleengine/hold.go:205

**Issue:** `lostPromptRetyped` is set before `typePendingPrompt`.
Every `*SessionBusyError` is raised before the paste (send.go:198, send.go:228), so a held retype typed nothing.
The latch still stops every later tick, and the error branch only logs.
A held reading such as the no-input-box hold from issue #137 leaves the strand idle with no task until its timeout.

### [MEDIUM] The no-input-box reading itself is not checked

**Location:** internal/shuttleengine/run.go#sendStartPrompt

**Issue:** The board entry describes the idle screen in its own text, and its first part says the reading "does not recognize this layout" as a possible cause.
The idle wait already polled for the whole `send_ready_timeout_s` window before this diff (send.go:209-232), and every reading in it was held.
The diff adds up to two more windows of the same reading.
A layout the reader misreads every time still fails the start, only later.
`_lym/landing/summary.md` says the cause is still unknown.
The run directories the entry names are gone, but the described layout can be checked against Claude's `inputBoxInterior` (claudeengine/session.go:300), which needs a top rule, a caret line and a bottom rule.

### [MEDIUM] The fresh-start retype is untested

**Location:** internal/shuttleengine/run.go:877

**Issue:** `TestWait_RetypesAPromptLineThatStartedNoTurn` reaches the retype only through `Attach` (attach_test.go:69).
Nothing drives `Start` then `Wait` with a dropped line, so removing run.go:877-879 fails no test.

### [MEDIUM] The busy retry's bounds are untested

**Location:** internal/shuttleengine/run.go:896

**Issue:** The new row turns idle after one window and asserts one typing.
The active-operator row is busy on every send, but it asserts only the error and the died record, so it passes with any retry count.
No row reaches the deadline stop.
The foreign-box row counts no pastes, so a retry widened to `errors.Is(err, ErrSessionBusy)` would paste the line twice and pass.

### [LOW] The new sends are not cut to the run deadline

**Location:** internal/shuttleengine/run.go:894

**Issue:** `run.newSendContext()` leaves `deadline` zero (send.go:92-106), so `awaitIdleSession` never cuts its limit to the run deadline (send.go:209-212).
A resend begun just before the deadline runs a full idle wait and submit window past it.
`Wait` calls `retypeLostPrompt` before its own deadline check (wait.go:313, wait.go:455).
The overrun is bounded by the retry count, so the impact is small.

### [LOW] The retype's guards are untested

**Location:** internal/shuttleengine/hold.go:190

**Issue:** The covered check, the `SessionSignalParser` check and the submit-window comparison at hold.go:190-198 each fail no test when removed.

### [LOW] The failure capture's branches are untested

**Location:** internal/shuttleengine/run.go:906

**Issue:** `startSendEngine` implements no `StyledBoxReader`, and no row makes the failure capture fail.
So neither the styled choice nor the fallback is pinned.

### [LOW] abandonStartup's doc names the wrong capture source

**Location:** internal/shuttleengine/wait.go:805

**Issue:** The doc says the saved capture is the one `checkLivenessTick` recorded.
After this diff, a failed prompt delivery saves the capture `keepFailureCapture` took (run.go:874, run.go:906-913).

### [NIT] promptDelivered's comment overstates the latch

**Location:** internal/shuttleengine/run.go:326

**Issue:** The latch now stops only `DeliverPendingPrompt`.
`retypeLostPrompt` types the line again through `typePendingPrompt`.

## Rejected

- code-correctness F3, a pause beginning during the retype stalls `Wait` inside `sendThroughPause`: `deliverPromptAtHold` and `deliverPromptOnAttach` already wait out the pause inside the tick in the same way, and PATTERN-reed-panes lists "the deferred prompt wait out the pause and resend" as that caller's rule.
- The assistant's BLOCKING severity on its three test findings, which cites PATTERN-test-speed: that entry pins tiers, speed and redundancy, not missing coverage. Missing coverage is graded by the rubric's test-economy bullet, so the findings are kept at MEDIUM (F4, F5) and LOW (F7, F8).
- code-tests LOW, only `*SessionBusyError` is retried and nothing pins it: merged into F5.
- The assistant's F4, new sends can outlive the run deadline: merged into F6.
- On the focus question whether a busy retry can type the line twice: not a defect. Every `*SessionBusyError` site comes before the paste, and the paste-time sentinels `ErrSentTextInDraft` and `ErrForeignBoxText` are not `*SessionBusyError` (send.go:585-591).
- On the focus question whether every caller of `typePendingPrompt` holds `sendMu` and `promptDelivered` is still set on every earlier path: not a defect. Both callers lock `sendMu`, the lock order is `sendMu` then `recordMu` as before, and `promptDelivered` is set exactly when a delivery typed.
