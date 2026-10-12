# Card 4 — guarded interrupt and guarded rewind-picker close in shuttle

**Create:**
- `internal/shuttleengine#ErrNoRunningTurn`
- `internal/shuttleengine#ErrRewindPickerOpen`
- `internal/shuttleengine#guardedInterrupt`
- `internal/shuttleengine#closeRewindPicker`
- `internal/shuttleengine#Runner.InterruptSettle`
- `internal/shuttleengine#Runner.PaneRunningTurn`
- `internal/shuttleengine#fakeEngine.RunningTurn`
- `internal/shuttleengine#fakeEngine.InterruptSettle`
- `internal/shuttleengine#pickerEngine`
- `internal/shuttleengine#pickerEngine.RewindPicker`
- `internal/shuttleengine#pickerOnlyEngine`
- `internal/shuttleengine#pickerOnlyEngine.RewindPicker`
- `internal/shuttleengine#TestGuardedPickerClose`

**Edit:**
- `internal/shuttleengine#Runner.Interrupt`
- `internal/shuttleengine#Run.Interrupt`
- `internal/shuttleengine#awaitIdleSession`
- `internal/shuttleengine#fakeEngine`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#fakeReed.CapturePane`
- `internal/shuttleengine#TestRun_Interrupt_PlaysEscape`
- `internal/shuttleengine/doc.go`
- `internal/shuttlecli#shuttleCLI.interruptCmd`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#InterruptGuard`
- `internal/shuttleengine#RewindPickerReader`
- `internal/shuttleengine#requireReadyAgentPane`
- `internal/shuttleengine#readinessEngine`
- `internal/shuttleengine#newFixture`

**Intent:**
D3, D4's settle and D8's close path: every interrupt key shuttle sends goes through a pane guard, and an open rewind picker is closed with one key and fails the operation.
Put the new production code in a new file `internal/shuttleengine/interruptguard.go` (package `shuttleengine`), with a file comment saying it holds the guarded interrupt and the guarded picker close.

The two sentinels:

- `ErrNoRunningTurn` means "nothing to stop".
  Callers wrap it with the strand: `fmt.Errorf("%w (strand %q)", ErrNoRunningTurn, guid)`, so the message keeps the way forward naming `lyx shuttle send`.
- `ErrRewindPickerOpen` is wrapped by the picker close with what happened: `"%w: strand %q: one interrupt key closed it; nothing was typed or interrupted, so read the pane before giving the session a turn"`, or, when the re-check shows no input box, `"%w: strand %q: one interrupt key was sent, but the picker or another dialog is still on screen; nothing more was sent, so close it in the pane, then retry"`.
  Neither sentinel wraps `ErrSessionBusy`, so no caller retries a picker as a busy session; the gate re-prompt in `wait.go`, the orch watcher's typed steps and the loom resume sends treat it as a failed send, which keeps the bug loud as D8 wants.

`closeRewindPicker(reed, engine, clock, guid, capture)`:

- When the engine does not implement `RewindPickerReader`, or `RewindPicker(capture)` is false, it sends nothing and returns nil.
- Otherwise it sends exactly one key: the first input of `engine.InterruptSequence()` that sets `Key`, through `playInputs` on a one-element slice.
  A send error returns wrapped.
- It then sleeps the settle on `clock` (the engine's `InterruptSettle` when it implements `InterruptGuard`, else 0) and captures the pane once more.
  A capture error, or `readBox` finding a box reader that reads no box, gives the "still on screen" message; anything else gives the "closed" message.
  Either way it returns the wrapped `ErrRewindPickerOpen` and sends nothing more, so no entry is ever selected and no key follows the close in the same operation.

`guardedInterrupt(reed, engine, clock, guid)`:

1. `requireReadyAgentPane` first, as today.
2. An engine that does not implement `InterruptGuard` fails with an error saying the engine has no running-turn reading, so no interrupt key is sent.
   This is a fail-loud wiring guard: every engine in the tree implements it after cards 2 to 4, so no test row reaches it.
3. One `reed.CapturePane`; a capture error returns `fmt.Errorf("shuttle: capture strand %q's pane before the interrupt: %w", guid, err)` and sends nothing, so the guard fails closed.
4. `closeRewindPicker` on that capture; a non-nil result is returned as is.
5. `RunningTurn(capture)` false returns the wrapped `ErrNoRunningTurn` and sends nothing.
6. Otherwise `playInputs` sends `engine.InterruptSequence()`.

Bound (D3): the guard only withholds a key, never adds one.
It lets through a key whose turn ends between the capture and the key, and a capture quoting the running-turn hint; either costs one key on an idle prompt, which opens nothing.
The close sends at most one key per detection, only on a capture the provider reads as the picker.

Wiring:

- `Runner.Interrupt` keeps its told-path and `FindRun` checks and then returns `guardedInterrupt(r.reed, r.engine, r.clock, guid)`.
- `Run.Interrupt` returns `guardedInterrupt(run.runner.reed, run.runner.engine, run.clock, run.state.StrandGUID)`.
- Both godocs say the interrupt goes only into a pane showing a running turn, closes an open rewind picker with one key and fails, and returns `ErrNoRunningTurn` when there is nothing to stop.
- `awaitIdleSession`: right after `requireReadyAgentPane` (the engine without `SessionCycler`) or `requireLiveStrand` (the engine with it), and before any readiness reading or operator-activity check, an engine that implements `RewindPickerReader` gets one `sc.reed.CapturePane`.
  On a successful capture, `closeRewindPicker(sc.reed, sc.engine, sc.clock, sc.guid, capture)` returns its error, which ends the send with nothing typed.
  A failed capture is handled per branch, so nothing is typed into an unread picker:
  - The engine without `SessionCycler` has no polls after it, so the failed capture fails closed: it returns `fmt.Errorf("shuttle: capture strand %q's pane before the send: %w", sc.guid, err)` and types nothing.
  - The engine with `SessionCycler` skips the close, and the idle wait goes on as today; its polls read the pane again and hold on a pane they cannot read or that shows no input box.
  An engine without the capability pays no extra capture, so the scripted capture queues of the existing send tests are unchanged.
  This covers every verified send: `Run.Send`, `Runner.Send`, the session commands and the skill-load and prompt-line sends.
- `Runner.InterruptSettle` returns the engine's `InterruptSettle` when it implements `InterruptGuard`, else 0, for the pause and the orch watcher to wait out (cards 8 and 9).
- `Runner.PaneRunningTurn(guid)` runs the told-path check, `FindRun` and `requireLiveStrand`, then captures once and answers `RunningTurn(capture)`.
  A capture error is returned, and an engine without `InterruptGuard` fails as in step 2.
  The pause reads it after its second key's settle (card 8).

Test support in `fakes_test.go`:

- `fakeEngine` gains a `Settle time.Duration` field and implements `InterruptGuard`: `RunningTurn` is `strings.Contains(capture, "RUNNING")` and `InterruptSettle` returns `Settle`.
  Its type comment names the marker; no existing capture in the package's tests contains it.
  Every engine fake that embeds `fakeEngine` inherits the guard, so `Run.Interrupt` through any of them needs a capture showing `RUNNING` to send a key.
- `pickerEngine` embeds `readinessEngine`, so it has the box reader and the idle reading, and implements `RewindPickerReader`: `RewindPicker` is `strings.Contains(capture, "REWIND-PICKER")`.
  Captures that show the box after a close carry the caret `"❯ "` that `readinessEngine.InputBoxText` reads.
- `pickerOnlyEngine` embeds `fakeEngine` and implements `RewindPicker` with the same needle, so it reads the picker but is not a `SessionCycler`; it drives the send path's branch without polls.
- `fakeReed` gains `CaptureErrQueue []error`, consumed FIFO by `CapturePane` with the last entry sticking, like `CaptureQueue`.
  A nil entry answers from `CaptureQueue` as today, and a non-nil entry fails that call; `CaptureErr` keeps failing every call.
  It lets a row pass the ready check on its first capture and fail the guard's capture after it.

Tests:

- `TestRun_Interrupt_PlaysEscape` becomes the guarded-interrupt table.
  Its doc comment says the guard sends the key only into a pane showing a running turn.
  Each row gains `viaRunner bool` (a `withStrand("strand-1")` fixture calling `Runner.Interrupt("strand-1")`; otherwise `newRun(Spec{})` and `Run.Interrupt`), `captures []string`, `captureErrs []error`, `wantKeys int` and `wantErr error`.
  Rows:
  - "ready pane showing a running turn": captures `["RUNNING"]`, one `Escape`, no text.
  - "ready probe retries a transient boot": script `[StartupPending, StartupReady]`, captures `["RUNNING"]`, one `Escape`.
  - "runner interrupt into a running turn": the first row with `viaRunner`, one `Escape`.
  - "a pane showing no running turn gets no key", for `Run` and for `Runner`: captures `["idle"]`, zero keys, `errors.Is(err, ErrNoRunningTurn)` and the message contains `lyx shuttle send`.
  - "a failed guard capture sends no key": `captureErrs [nil, boom]`, zero keys, the error wraps `boom` and is not `ErrNoRunningTurn`.
    The first capture passes `requireReadyAgentPane`, so the second is the guard's own.
  
  Every `viaRunner` row's engine sets `Settle: 2 * time.Second`, and after its `Interrupt` assertions the row also asserts the two new Runner methods on the same fixture.
  `runner.InterruptSettle()` must equal 2 s.
  `runner.PaneRunningTurn("strand-1")` reads the sticky last capture: true on the running rows, false on the no-running-turn row, and an error wrapping `boom` on a capture-failure row run with `viaRunner`.
  So the capture-failure row runs for `Run` and for `Runner`.
- New `TestGuardedPickerClose` in `run_test.go`, one table with `op` in {`run-interrupt`, `runner-interrupt`, `runner-send`} over a `pickerEngine` whose `StartupScript` is `[StartupReady]` and whose `Settle` is 2 s.
  Each row uses a fake clock, a `withStrand("strand-1")` fixture for the runner ops, and `Config{SendReadyTimeoutS: 5}`.
  Rows:
  - "a picker on an interrupt is closed with one key and fails", for both interrupt ops: captures `["REWIND-PICKER", "REWIND-PICKER", "❯ "]`, since `requireReadyAgentPane` takes the first capture before the guard's own.
    Exactly one `SendKey` `Escape`, no `SendText` or `PasteSubmit`, the clock advanced by at least 2 s, the call log's last entry a `CapturePane` after the `SendKey:Escape`.
    `errors.Is(err, ErrRewindPickerOpen)`, the message contains "closed it", and `errors.Is(err, ErrSessionBusy)` is false.
  - "a picker on a send is closed with one key, fails and types nothing": captures `["REWIND-PICKER", "❯ "]` through `Runner.Send("strand-1", "hello")`, since the send path's first capture is the picker check's own; the same assertions, and no typed or pasted text.
  - "a picker still on screen after the close sends nothing more", for `runner-interrupt` and `runner-send`: captures `["REWIND-PICKER"]`, which sticks and shows no caret.
    Exactly one `Escape`, and the message contains "still on screen".
  - "no picker on an interrupt: the close sends nothing and the guard decides": captures `["RUNNING ❯ "]`, exactly one `Escape`, the guarded interrupt's own, and a nil error.
  - "no picker on a send: the close sends nothing and the send goes on to its idle wait": captures `["working"]`, so the idle wait holds on a pane that is not idle until the 5-second window runs out on the fake clock.
    It expects `ErrSessionBusy` and zero `SendKey` calls.
  - "a failed picker capture on a send skips the close and the idle wait holds": `runner-send` with `captureErrs [boom, nil]` and captures `["working"]`, so the picker check's capture fails and the polls read a pane that is not idle.
    It expects `ErrSessionBusy`, zero `SendKey` calls and no typed or pasted text.
  - "a failed picker capture on a send without polls fails closed": `runner-send` over a `pickerOnlyEngine` with `StartupScript` `[StartupReady]`, `captureErrs [nil, boom]` and captures `["❯ "]`, since `requireReadyAgentPane` takes the first capture before the picker check's own.
    The error wraps `boom` and is not `ErrSessionBusy`, with zero `SendKey` calls and no typed or pasted text.
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/shuttleengine` from 168 to 169 for `TestGuardedPickerClose`.

The other tests that call `Interrupt` keep passing unchanged:

- `TestRun_InterruptAndSend_RefuseDeadOrUntrackedStrand` and `TestRun_InterruptAndSend_RefuseAgentlessShellPane` refuse before the guard's capture.
- The told-path tests at the top of `run_test.go` refuse at the told error.
- `internal/webstercli`'s wiring test refuses at `FindRun`.
- The `llm` smoke test `TestSmokeInterruptSendContinues` interrupts a real mid-turn pane, which shows the running-turn hint.

`internal/shuttleengine/doc.go` gains a paragraph after the one on the verified send's idle wait.
It covers the guarded interrupt and its fail-closed capture; `ErrNoRunningTurn` as "nothing to stop"; the picker close before every interrupt and every verified send, with its one key, its settle and its re-check; `ErrRewindPickerOpen`; and the two new Runner methods.

`lyx shuttle interrupt`'s help, in the same commit as the behavior (`internal/shuttlecli`'s `interruptCmd`; its `RunE` body is unchanged):

- `Long` says the interrupt is sent only when the pane shows a running turn.
  On a pane with no running turn it sends nothing and fails, and `lyx shuttle send <guid> <text>` gives the agent a turn instead.
  On a pane showing the rewind picker it sends one interrupt key to close the picker and fails, with nothing interrupted.
- `Short` becomes "stop a shuttle run's running turn without killing its pane".
- `TestRunCLI_PositionalArgValidation` does not read the help text.

**ImpactSummary:** `Runner.Interrupt` and `Run.Interrupt` now refuse a pane with no running turn, which changes `lyx shuttle interrupt` (help here, CLI test in card 7), the pause (card 8) and the orch watcher (card 9). Verified sends gain one capture only for an engine that reads the picker, which is Claude and shuttlefake in production and tests.

**Commit:** 4: Guard every interrupt key by the pane and close an open rewind picker once
