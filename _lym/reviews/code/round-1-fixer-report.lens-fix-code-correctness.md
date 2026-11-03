```yaml
lens: fix-code-correctness
outcome: reported
```

Task: the fix-code-correctness lens over the F1–F6 change set (commits c5a961b2..93d583d8).

Lens list: none asked for.

Findings:

1. **LOW** — `gate_noinputbox_test.go`, row "a delivered re-prompt and the next arrival restart the count".
   - The row claims to pin the reset on a delivered re-prompt (`wait.go:582`), but it passes with only that reset removed.
   - The arrival reset (`wait.go:347`) restarts the count too, because the next re-prompt can only follow an arrival.
   - Evidence: the fixer's mutation run removed only the 347 reset (`wait.go:347`), and this row still passed, as the arrival row alone failed.
   - The row fails only when both resets are gone. The 582 reset is effectively unpinned. The row name and comment overstate what it pins.
   - Suggested fix: reword the row to say it covers the pair.

2. **LOW** — `pattern/PATTERN-pane-text-injection.md`, line 3 reword ("closes a menu only past the quiet-window guard: no operator was active in the pane within `operator_quiet_s`").
   - The sentence is general, but the PATTERN's own bullets for the guarded picker close name no operator-activity check, only the ready-agent-pane check and the capture.
   - Only the new dismissal is shown to check operator activity (`send.go` `dismissPicker`).
   - If the rewind close does not check it, the sentence is false for that path.
   - Suggested fix: scope the sentence to the dismissal, or confirm the close checks activity.

Checked, no defect:
- **`busyHold.noInputBox`:**
  - Every `busyHold` literal uses named fields, so all callers hold.
  - `awaitIdleSession` ORs the field with the reason constant.
  - A failed dismissal key (`SendKey` error) sets `picker.tried` and returns a hold with `noInputBox`, so the next poll reads normally.
- **`TestAwaitIdleSession_FailedDismissalKeyHoldsAsNoInputBox`:**
  - It fails with the defect in place. With `deadline == now`, the first poll is the last, so the reason would be the "dismissing … failed" text and `NoInputBox` would be false.
  - The gate row "key cannot be sent" passes with the defect in place, because the last poll's reason is the no-box one. That is fine, since the unit test discriminates.
- **`TestDismissPicker_Guards`:**
  - Removing the `RunningTurn` guard makes the running row send an Escape and fail.
  - Removing the settle sleep makes the quiet row fail on the exact clock check.
- **`NoInputBox` rows and the conditioned-send assertion:**
  - The fixer's mutation run (removing the `boundaryIdle` reset, the arrival reset and the else-branch reset) failed the alternating, arrival and turn-start rows, as expected.
  - The `NoInputBox` assertion on a conditioned send fails if `conditionedsend.go:71` drops the field.
- **F6 fixture:** with the `─` rule above the options and none below, `inputBoxInterior` finds no box, so `TestDismissiblePicker` pins the intended shape.

Tool calls made: none; the context already held every file in its changed form.
