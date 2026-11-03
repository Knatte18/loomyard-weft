```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or say none.

**Per behaviour**
1. Picker recognised only when every needle is on screen and no input box is: `TestDismissiblePicker` (`claudeengine/session_test.go:141`). The fixture row pins the match. The "one phrase alone" row pins all-needles-required. The "transcript quoting the picker above an empty box" row pins the no-box condition. The "rewind picker" row pins that the table does not match the rewind picker. The "idle empty box" row is not distinct from the transcript row.
2. Escape sent and the re-prompt delivered: `TestGate_RepromptOverPicker`, row "a known picker is dismissed and the re-prompt delivered" (`gate_test.go:1279`). It asserts 1 send, 1 attempt, 1 Escape.
3. At most one dismissal per send: the "known picker its key does not close" row, with a sticky picker and `wantEscapes: maxNoInputBoxReprompts`. Removing `picker.tried` would give more Escapes per send and fail it.
4. Operator-activity guard: the "operator active over a known picker" row asserts 0 Escapes, 0 sends and a run held until the deadline.
5. Running-turn guard (`guard.RunningTurn(capture)` returning `handled=false`): none. `dismissPickerEngine` has no `InterruptGuard` and no row shows a running turn over a picker. Deleting that branch would break no test.
6. Interrupt-settle sleep after the key (`send.go:364`): none. No fake engine implements `InterruptGuard` with the picker.
7. Key-send failure path ("dismissing the provider's … picker failed"): none.
8. `NoInputBox` set at the idle-wait exhaustion (`send.go:209`): pinned indirectly. The sticky-picker and unknown-picker rows only reach the bound if it is set.
9. `NoInputBox` set in `conditionedReading` (`conditionedsend.go:71`): none. No test asserts it on a conditioned send's busy error.
10. Bound ends the loop after `maxNoInputBoxReprompts` holds, finalizing done with attempts spent: the sticky and unknown-picker rows. They assert 0 attempts, the single bound WARN and the end before 30 s.
11. A held send that is not `NoInputBox` resets the count (`wait.go:558`): none. No row mixes no-input-box holds with other holds. Removing the `else` reset would break no test.
12. Resets at a new arrival (`wait.go:347`), in `boundaryIdle` (`send.go:919`) and on a delivered re-prompt (`wait.go:582`): none. No test shows a count carried across a boundary or a delivery.
13. Unknown picker gets no key: the "an unknown picker ends the loop after the bound with no key" row asserts 0 Escapes.
14. `escapeKey` constant in `startup.go`: a pure refactor with no new behaviour. The existing interrupt tests cover it.

**Findings**
- MEDIUM, design, `internal/shuttleengine/send.go:352-355`: the running-turn guard in `dismissPicker` has no test row. The guard stops an Escape from interrupting a turn, so it needs a case. Add a fake engine that implements `InterruptGuard` with `RunningTurn` true over the picker. Assert 0 Escapes and a hold. Evidence: `dismissPickerEngine` at `fakes_test.go:996` and the rows at `gate_test.go:1275-1291`.
- MEDIUM, design, `internal/shuttleengine/wait.go:554-559`: the count reset on a held send that is not `NoInputBox` has no test, so the "in a row" semantics are unpinned. Add a row that alternates a no-input-box hold with another busy hold and asserts the loop is not ended. Evidence: the rows at `gate_test.go:1275-1291`.
- LOW, design, `internal/shuttleengine/wait.go:347`, `send.go:919`, `wait.go:582`: the other resets of `noBoxReprompts` (new arrival, `boundaryIdle`, delivered re-prompt) have no test, so a stale count could survive. Add at least one row where the count builds, an arrival or delivery occurs, and the bound then needs a fresh `maxNoInputBoxReprompts` holds.
- LOW, design, `internal/shuttleengine/send.go:360-362`: the failed-key-send path (a hold reason, and `tried` already set) and the settle sleep have no test. Add a row with a `SendKey` error that asserts the hold and no second try.
- LOW, design, `internal/shuttleengine/conditionedsend.go:71`: `NoInputBox` on the conditioned-send busy error has no assertion. Add a case in the conditioned-send tests, or drop the field there if nothing reads it.
- NIT, consistency, `claudeengine/session_test.go:158`: the "idle empty box" row repeats the transcript row. Both capture a live box, and the idle capture has no wording, so it fails the needle check regardless of the box check. Drop it or keep only the transcript row.

Tool calls: none; my context already held every diff hunk and test I needed.
