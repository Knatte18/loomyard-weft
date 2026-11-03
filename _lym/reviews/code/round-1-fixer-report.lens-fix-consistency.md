```yaml
lens: fix-consistency
outcome: reported
```

Findings for lens fix-consistency:

**MEDIUM** — `pattern/PATTERN-pane-text-injection.md`, line 3 (F1 reword)
- The new header says lym "closes a menu only past the quiet-window guard: no operator was active in the pane within `operator_quiet_s`."
- That claim is false for the guarded rewind-picker close, which the same document still lists as an exception.
- `closePickerBeforeSend` (`interruptguard.go:153`) calls `closeRewindPicker`, which plays the key after only a `RewindPicker(capture)` check.
- It makes no `OperatorActive` or `operatorActivityHold` call.
- The doc's own picker-close bullet names only the "capture the provider reads as the rewind picker" condition, so the header contradicts its sibling bullet.
- The trust-prompt dismissal also closes a dialog with no quiet-window check.
- The reword should scope the quiet-window claim to the picker dismissal, or the document should say what protects an operator's open rewind picker.

**LOW** — `pattern/PATTERN-pane-text-injection.md`, "Guarded interrupt" opener, "only through shuttle's pane guards"
- The "two" is dropped, and the picker-close bullet now says "one of two exceptions".
- The opener no longer says which guards it means, and the guarded interrupt bullets name the ready-pane check and the running-turn capture.
- A reader can no longer count the guards, and the opener and the "two exceptions" wording now disagree on what is counted.

I found no other disagreement between the changed text and its siblings:
- The `doc.go` NoInputBox and dismissal paragraphs still hold with the new `busyHold.noInputBox`.
- The `wait.go` and `doc.go` bound text matches the code.
- The `SessionBusyError` doc comment is still accurate.
- The test names cited in the PATTERN file exist.

Calls made:
1. `grep -rn "func closePickerBeforeSend" -A40 internal/shuttleengine/*.go`: my context lacked `interruptguard.go`, and I needed to know whether the rewind-picker close checks operator activity.
