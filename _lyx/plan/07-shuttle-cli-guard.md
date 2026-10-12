# Card 7 — `lyx shuttle interrupt` and `send` report the guards' failures

**Create:**
- `plan:internal/shuttlecli#TestInterruptAndSendCmd_FailWithTheGuardsMessage` -> `func TestInterruptAndSendCmd_FailWithTheGuardsMessage(t *testing.T)`

**Edit:**
- `internal/shuttlecli#shuttleCLI.interruptCmd`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttlecli#shuttleCLI.sendCmd`
- `internal/testkit/shuttlefake#Engine`
- `plan:internal/shuttleengine#ErrNoRunningTurn`
- `plan:internal/shuttleengine#ErrRewindPickerOpen`

**Intent:**
D3 and D8 at the CLI: `lyx shuttle interrupt` on a session with no running turn now fails instead of sending a key, and both verbs fail with the picker error's message when they find the rewind picker open.

- `interruptCmd`'s `Long` help says the interrupt is sent only when the pane shows a running turn.
  On a pane with no running turn it sends nothing and fails, and `lyx shuttle send <guid> <text>` gives the agent a turn instead.
  On a pane showing the rewind picker it sends one interrupt key to close the picker and fails, with nothing interrupted.
  `Short` becomes "stop a shuttle run's running turn without killing its pane".
  The `RunE` body is unchanged: it already prints `err.Error()` in the error envelope, so the message is the engine's.
- New `TestInterruptAndSendCmd_FailWithTheGuardsMessage` in `cli_test.go`, built like `TestSendCmd_AppendsMessageTail`.
  Each row starts a run with `newFakeClockRunner` over a `shuttlefake.Reed` and a `shuttlefake.Engine` whose `InterruptSequenceFn` answers one `Escape`.
  The reed's `CapturePaneFn` returns `"PICKER"` while a test-local `pickerOpen` flag is set and `"idle"` otherwise, and its `SendKeyFn` clears the flag on `Escape`.
  The engine's `RewindPickerFn` reads `"PICKER"` as the picker and its `RunningTurnFn` answers false.
  The flag is set only after `runner.Start` returns, so the start's own sends see no picker.
  Rows:
  - "interrupt on a pane with no running turn fails naming lyx shuttle send": `interruptCmd` with the run's guid.
    The envelope's `ok` is false, its `error` contains `lyx shuttle send`, and `SendKeyCalls` is empty.
  - "interrupt on an open picker closes it once and fails": the flag set, then `interruptCmd`.
    `ok` is false, `error` contains "rewind picker was open", and `SendKeyCalls` holds exactly one `Escape`.
  - "send on an open picker closes it once, fails and types nothing": the flag set, then `sendCmd` with "read the plan".
    `ok` is false, `error` contains "rewind picker was open", `SendKeyCalls` holds exactly one `Escape`, and `SendTextCalls` and `PasteSubmitCalls` are empty.
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/shuttlecli` from 25 to 26.

**ImpactSummary:** Help text and one new test; the verb's code path is unchanged, and `TestRunCLI_PositionalArgValidation` does not read the help text.

**Commit:** 7: Say in lyx shuttle interrupt's help what the pane guards refuse
