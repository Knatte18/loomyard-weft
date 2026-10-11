---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/01-claude-lone-eol-mark.md:49
    summary: Card 1 leaves the fake reed's mid-send capture unspecified and keys its switch-back on landing, which the capture itself decides; serving the lone-% capture throughout already lands every paste
---

### [LOW] Card 1's fake-reed capture sequence has an unspecified, circular middle state

**Location:** `_lyx/plan/01-claude-lone-eol-mark.md:49`

**Issue:**
The card says the reed's capture serves `pane-idle-eol-mark.txt` until a send arrives, "then whatever the test's paste hook needs for the send to land", and the lone-`%` capture again "once that send has landed".
Whether a paste has landed is decided by reading that same capture.
Claude's `PasteSubmits()` is true (`pasteSubmitsOnEnter`), so `pasteVerified` goes straight to `confirmPasted`, which returns nil when the box reads empty and sends a guarded Enter when the box holds exactly the sent text (`internal/shuttleengine/send.go`, `confirmPasted`).
With card 1's own reading, the lone-`%` capture reads as an empty box, so serving it on every capture lands each paste on the first confirm read.
Read another way, the middle state is a capture whose box holds the sent text, with the switch back triggered by "landed".
Then nothing returns the capture to an empty box: `confirmPasted` sends a guarded Enter and reads the same box again until the submit window closes.
The skill-load send's `ErrSubmissionNotLanded` then tears the start down through `abandonStartup`.
The pre-write guard question is answered either way.
Each guard runs before its paste or after the previous send returned, and so reads the lone-`%` box: `busyReading` falls back to `IdleSession` on an unknown session state, and `readBox` reads `InputBoxText` otherwise.
The defect is the undefined middle state, which a test writer has to work out from the send code.

**Fix:**
Replace the three-phase sequence with one sentence: the capture serves `pane-idle-eol-mark.txt` on every read, so each send's pre-write guard reads the lone-`%` box as idle and each paste's confirm read sees it as an empty box, which lands the send.
Alternatively, name the middle capture as the box holding the sent text, with the switch back to the lone-`%` capture on the guarded `Enter` key.

## Focus verdicts

**Card 3: does each no-tail row hold a non-empty capture?**
Yes for both rows; not a defect.
- Skill-load death: `skillReed.CapturePane` (`fakes_test.go`) echoes only the texts sent so far, and the startup probe runs before any send, so today the row records an empty `lastStartupCapture`.
  The planned `Screen` field makes that capture non-empty.
  `checkLivenessTick` set `run.state.Started` on the ready probe, and `loadSkillsThenPrompt` calls `abandonStartup(OutcomeDied, nil)` on `died`, so the `Started` check alone decides the no-tail assertion.
  `skillFakeEngine` reads no input box, and `sendVerified` checks only the needle count, so an extra first line does not disturb the sends.
- Reed-status retry: with `passFirst: 1`, the first `Status` comes from the inner reed, which shows the strand live.
  `checkLivenessTick` then records the non-empty queued capture, and `fakeEngine.Startup` with no script reads `StartupPending`.
  The next `maxStatusRetries` calls fail, and the tick cap (`ceil(30s/1s)+1+maxStatusRetries`) and the 30 s window both leave room for those ticks.
  The exit carries a cause with `Started` false and a non-empty capture, so the cause check decides.

**Card 4: does an existing test already drive a gate that never clears?**
Yes, and the card now extends it; not a defect.
`TestStartup_UndismissableGateUntilWindowExpires` (`startup_test.go`) is the `awaitStartup` gate-never-clears test.
Card 4 turns it into the table and states that the package gains no top-level test, so `internal/shuttleengine` stays at 160 against its budget row of 161.
`TestRun_Wait_StartupWindow`'s "trust prompt that never clears" row (`wait_test.go`) drives `Wait`'s probe, not `awaitStartup`, and asserts neither key count nor play times, so it does not cover the new rows.
It also survives the throttle.
Under `shortStartupConfig` the gap is 1 s and `Wait` probes 1 s apart on a fake clock that advances, which the at-or-past rule admits.

**Card 1: does each send's pre-write guard see the lone-`%` box?**
Yes; the gap is only how the middle capture is specified, filed as F1.
The color send goes `typeColor` → `sendSequence` → `sendVerified`, because Claude is a `TextPaster`, an `InputBoxReader` and a `SessionCycler`.
The skill-load send goes `loadSkillTurn` → `sendVerified`.
Both run `awaitIdleSession` → `busyReading` on a fresh capture before their paste, and the second guard runs only after the first send returned.
