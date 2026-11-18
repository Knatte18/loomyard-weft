```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code and tests through the fix-code-correctness lens.

**Findings**

1. **LOW** — `hold.go` `retypeLostPrompt`, the new `!run.clock.Now().Before(run.deadline)` guard (F6). No test fails with that guard removed.
   - The only deadline rows are in `TestStartup_SendFailuresTearDown`, and they cover the start's resend loop.
   - In `TestWait_RetypesAPromptLineThatStartedNoTurn`, the timeout row (`window / 2`) is already stopped by the submit-window check. A retype past the deadline never occurs in any row, so the guard is untested.
   - Evidence: the retype only fires once the window has elapsed, and no Wait row has a deadline that falls after the window and before the retype.

2. **LOW** — `hold.go` `retypeLostPrompt`, the busy branch.
   - `logger.Warn` fires on every tick the retype is held busy.
   - For an engine without `SessionCycler`, the hold comes from `operatorActivityHold` and returns at once with no idle wait.
   - An operator active in the pane for a minute therefore logs about once per poll interval, each at Warn.
   - Evidence: in `awaitIdleSession`, the `!ok` branch returns the `SessionBusyError` without sleeping.
   - The hold costs no time, so the Warn adds log volume only. Info, or a once-per-episode log, would avoid it.

**Checked, no defect**

- `typePendingPrompt`: `promptSendOffset` is set only after a successful send, under `sendMu`. A failed send leaves the old pair, so a failed retype cannot move the watch.
- `retypeLostPrompt` latch:
  - A busy error leaves `lostPromptRetyped` unset.
  - Other errors and a `(false, nil)` result latch it.
  - `errors.As` sees the `*SessionBusyError` through the `fmt.Errorf("%w")` wrap in `typePendingPrompt`.
- `sendStartPrompt` and `sendThroughPause`: the deadline carries through a pause resend. A zero deadline stays zero for every other caller.
- Tests that fail with the defect in place:
  - F1 (operator turn seeded before the typing): reverting to `PromptOffset` types once.
  - F2 (busy retype): the old latch-before-send types once.
  - The pause, no-signals and timeout rows each fail without their guard.
  - F4 fails without the `promptSentAt` assignment in `loadSkillsThenPrompt`.
  - The F5 wait counts, the foreign-box paste count and the F8 styled and boot-capture rows are pinned.
  - The reasoning for the F8 boot-capture row is that a failed capture must leave the boot capture in place.

No tool calls were made: my context already held every file I needed.
