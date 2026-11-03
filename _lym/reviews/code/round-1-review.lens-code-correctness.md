```yaml
lens: code-correctness
outcome: reported
```

Lens restated: code-correctness of the diff's new and changed branches. Each line below is the case I ran and the verdict.

**List**
- `dismissPicker`, picker shown, no running turn, operator idle: `tried` is set, Escape is played, the pane settles 2 s, then one more `busyReading` runs. With `tried` already true that reading cannot re-enter `dismissPicker`, so it cannot loop. Correct.
- `dismissPicker`, operator active: it returns a hold without a key and without setting `tried`, so a later poll retries. Correct. The hold reason differs from `readinessReasonNoInputBox`, so `NoInputBox` stays false and the count resets (this is also what the "operator active" gate test expects).
- `dismissPicker`, `playInputs` fails: `tried` is already true, so it holds once and never retries within this send. This matches the documented "once per send". Correct. The reason is not the no-input-box constant, so the count resets (see F2).
- `dismissPicker`, `RunningTurn` capture: it returns `handled=false` and the normal hold follows. Correct.
- `dismissPicker`, unknown readiness (`reading.unknown`, ready false): it is still reached and works from the pane alone. Correct.
- `busyReading` stray-clear path: `picker` is threaded through `clearStrayCharacter`. `tried` is untouched by the stray path, and the stray path is not skipped. Correct.
- `awaitIdleSession` timeout: `NoInputBox` equals `busy.reason == readinessReasonNoInputBox`. `judgeReadiness` returns that reason only for `box.reader && !box.read`. The pane-fallback reason "the pane shows no input-ready provider" is deliberately not flagged. A gated run always has a run record, so it takes the readiness path. Correct.
- `handleGatedBoundary`, five holds in a row with `NoInputBox`: the count reaches 5 and the run finalizes `OutcomeDone` with the attempts spent. The terminal path next to it (`wait.go:577-579`) also finalizes `OutcomeDone`, so it is consistent. Correct.
- `handleGatedBoundary`, a mix of hold kinds: any other busy, `ErrSubmissionNotLanded` or `ErrPaused` error resets the count to 0, so alternating kinds can defeat the bound. See F2.
- `handleGatedBoundary`, successful re-prompt: it resets the count. Correct.
- `noBoxReprompts` reset sites: the only sites that touch `unsentReprompt` (`wait.go:346`, `wait.go:581`, `send.go:918`) all reset the count too. The set sites need none. Correct. The `run.go` comment is accurate.
- `DismissiblePicker`, fixture capture: the caret line `❯ 1.` has no `─` rule above (the fixture uses `▔`) and none below, so `inputBoxInterior` returns no box. Both needles match after normalising, so it is a picker. Correct.
- `DismissiblePicker`, transcript quote above a live box: the box is found, so it is not a picker. Correct. One needle alone: no match. Correct.
- `conditionedReading`: it sets `NoInputBox` and does not dismiss. Correct, and that is its existing contract.
- `escapeKey` constant in `InterruptSequence`: value unchanged. Correct.

**Findings**
- **F1: LOW, design.** `send.go:343-368`, evidence `send.go:353` and `send.go:365`. The Escape gate is the pane-text `RunningTurn` check only. When the hook session state says a turn is running but a picker has hidden the "esc to interrupt" hint, Escape still goes out. The picker would most likely consume the key, but the code does not prove that.
- **F2: LOW, design.** `wait.go:554-559` and `send.go:356-361`. A failed dismissal key, or an active operator, produces a hold that is not `NoInputBox`. That resets `noBoxReprompts`, so a picker held up by an operator or a failing key send is never bounded and still waits for the run deadline. The board entry asks for a bounded re-prompt for any picker shuttle does not clear. The fix is to keep `NoInputBox` set on the two `dismissPicker` hold reasons, or to compute it from the pane capture rather than the reason string.
- **F3: NIT, design.** `send.go:364-366`. After the key, `guard.InterruptSettle()` (2 s) is slept without being cut to `limit` or `sc.deadline`. A send whose window or run deadline ends in under 2 s overshoots it by up to 2 s. The sleep should be `min(settle, limit.Sub(now))`.
- **F4: LOW, consistency.** `claudeengine/testdata/pane-autocompact-picker.txt`. The fixture is hand-written, with a `▔` rule and invented body text, not a capture from Claude Code. If the real picker draws a `─` rule above and a rule below its options, `inputBoxInterior` would find a "box" and `DismissiblePicker` would not match. I could not check this against a real capture.

**Tool calls:** none. The manifest already held every definition and caller I needed.
