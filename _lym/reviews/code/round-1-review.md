---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/shuttleengine/send.go:343
    summary: dismissPicker sends Escape, the provider's interrupt key, through a path PATTERN-pane-text-injection does not allow, with no PATTERN change and no Pattern-Departure trailer
    fix: In pattern/PATTERN-pane-text-injection.md add a bullet naming the guarded picker dismissal as a second bounded key exception beside the guarded picker close. State its guards (the engine names the keys, a capture the engine reads as a known picker with no input box, no running turn, no operator active in the quiet window, once per send attempt, an interrupt settle after the key). Add the dismissal to the "Explicit key deliveries stay outside the rule" list and reword the "never closes a menu the operator has open" line to say the quiet-window guard is what protects an operator's open menu. Then adjust the PATTERN.md `PATTERN-pane-text-injection` line ("an interrupt key only via shuttle's guards") so it names the dismissal too. Land these edits in this same diff, or put a Pattern-Departure trailer on the commit that names the entry and gives the reason.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/shuttleengine/send.go:352
    summary: the running-turn guard in dismissPicker has no test row, so deleting it breaks no test although it is the only thing keeping Escape from interrupting a turn
    fix: In internal/shuttleengine/fakes_test.go add a dismiss-picker fake that also implements InterruptGuard, with RunningTurn true for a chosen capture and a short InterruptSettle. Add a TestGate_RepromptOverPicker row, or a TestDismissPicker row, where the pane shows the picker and a running turn. Assert zero Escape keys, a hold, and that picker.tried stays unset so a later poll can try again.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/shuttleengine/wait.go:554
    summary: the in-a-row semantics of noBoxReprompts are unpinned; the reset on a held send of another kind and the resets on a new arrival, in boundaryIdle and on a delivered re-prompt have no test
    fix: In gate_test.go add rows to TestGate_RepromptOverPicker, or a sibling table test, that (a) alternate a no-input-box hold with a hold of another kind and assert the loop is not ended at maxNoInputBoxReprompts total holds, and (b) build the count, then deliver the re-prompt or start a new boundary, and assert the bound needs a fresh maxNoInputBoxReprompts holds. Cover these through the public Wait path with the existing fake clock.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/send.go:360
    summary: a failed dismissal key sets a hold that is not NoInputBox, which resets noBoxReprompts, so a known picker whose key cannot be sent is never bounded and waits for the run deadline
    fix: In dismissPicker, when playInputs fails, return the hold with the dismissal-failed reason and set a new busyHold field noInputBox true. Have awaitIdleSession set NoInputBox from that field or from the reason constant, instead of comparing the reason string alone. Leave the active-operator hold unbounded, since TestGate_RepromptOverPicker pins that. Add a row with a SendKey error that asserts the loop ends after the bound and that one key was tried per send.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/conditionedsend.go:71
    summary: conditionedReading sets SessionBusyError.NoInputBox, but no code reads it on that path and no test asserts it
    fix: Add an assertion to the existing conditioned-send busy-error test that a pane with no input box yields NoInputBox true, which also fits the SessionBusyError doc comment. If the field is not wanted on a conditioned send, remove the assignment at conditionedsend.go:71 and narrow the doc comment at send.go:23 and shuttleengine/doc.go:345 to verified sends.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/shuttleengine/claudeengine/testdata/pane-autocompact-picker.txt:1
    summary: the fixture is hand-written with a ▔ rule glyph and invented option text, and its glyph is what keeps inputBoxInterior from finding a box, so the test does not pin the shape Claude draws
    fix: Rewrite the fixture so its rule line uses the ─ glyph the repo's other dialog and picker fixtures use. Keep the wording lines from the board entry. If a live capture of the picker is available, replace the fixture with it. Check that TestDismissiblePicker still reads it as a picker, with no bottom rule under the options, and add a note in the summary that it is not a live capture.
---

### [BLOCKING] Escape dismissal is outside PATTERN-pane-text-injection

**Location:** internal/shuttleengine/send.go:343
**Issue:** `PATTERN-pane-text-injection` says an interrupt key reaches an agent pane only through shuttle's two pane guards, and names the guarded close of the rewind picker as the one exception. Its list of explicit key deliveries does not contain a picker dismissal, and its first line says lym never closes a menu the operator has open. `dismissPicker` plays the engine's keys, which for Claude are `Escape`, the same key `InterruptSequence` returns, through `playInputs` after only a `RunningTurn` check and an operator-activity check. It skips the ready-agent-pane check and the re-capture the guarded close does. The behaviour is what the board entry asks for, so it is not wrong, but it adds a key-delivery path the rule document does not allow. The diff has no `PATTERN.md` or `pattern/` edit and the commit has no `Pattern-Departure:` trailer. The change description says "Escape is also Claude's interrupt key", so the author saw the conflict.

### [MEDIUM] Running-turn guard of the dismissal is untested

**Location:** internal/shuttleengine/send.go:352
**Issue:** `dismissPickerEngine` has no `InterruptGuard`, so no `TestGate_RepromptOverPicker` row reaches the `guard.RunningTurn(capture)` branch or the settle sleep after the key. The guard is the only thing that keeps the dismissal from interrupting a running turn. The rubric treats a behaviour the diff adds with no coverage as a finding.

### [MEDIUM] Reset semantics of noBoxReprompts are untested

**Location:** internal/shuttleengine/wait.go:554
**Issue:** The bound is stated as consecutive holds. The `else` branch at `wait.go:558` that resets the count on a hold of another kind, the reset in `Wait` on a new arrival (`wait.go:347`), the one in `boundaryIdle` (`send.go:919`) and the one on a delivered re-prompt (`wait.go:582`) are each pinned by no row. Each of the four could be deleted with the tests staying green, and a stale count would then end a run early.

### [LOW] A failed dismissal key leaves a picker unbounded

**Location:** internal/shuttleengine/send.go:360
**Issue:** `NoInputBox` is derived by comparing the reason string with `readinessReasonNoInputBox`. The dismissal-failed hold carries a different reason, so it resets the count at `wait.go:558`. A known picker whose Escape cannot be sent is then retried every tick until the run deadline, which is the stall the board entry reports. The active-operator hold is unbounded on purpose, and a test pins it, so only the key-failure hold is a defect.

### [LOW] NoInputBox on a conditioned send is set but never read or tested

**Location:** internal/shuttleengine/conditionedsend.go:71
**Issue:** The only reader of `SessionBusyError.NoInputBox` is `handleGatedBoundary`, which sends through `sendWithin` and never through a conditioned send. The assignment is a behaviour with no consumer and no assertion.

### [LOW] Fixture does not pin the shape Claude draws

**Location:** internal/shuttleengine/claudeengine/testdata/pane-autocompact-picker.txt:1
**Issue:** The change description says the fixture is drawn from the board entry and not transcribed from a live pane. It draws its rule with `▔`, which `isBoxRule` does not accept, and that is why `inputBoxInterior` finds no box. The board entry gives only the wording. If the real picker draws a `─` rule above its options, the same result still holds only while no `─` rule sits below them. The test therefore pins an invented shape.

## Rejected

- code-correctness F1 (Escape sent while the hook state reports a running turn and the picker hides the hint): speculative. The picker has input focus, so its own Escape binding takes the key, and the `RunningTurn` and operator guards already apply; no input shows an interrupt.
- code-correctness F3 (the settle sleep after the key is not cut to the send's limit): the same uncut settle is how `clearStrayCharacter` sleeps `SubmitSettle`, so it follows the existing convention, and the overshoot is bounded by 2 s.
- code-tests NIT (the "idle empty box" row repeats the transcript row): the rows differ. The idle row has a live box and none of the wording, the transcript row has a live box and the wording, so the two pin different inputs; no existing test covers the same input.
- code-affected-set LOW (other `"Escape"` literals in claudeengine against the new `escapeKey` constant): a grep of production `.go` files finds no other `"Escape"` literal in claudeengine, so there is nothing to fix.
- Dismissal in every verified send's idle wait rather than only the gate re-prompt: licensed. The board entry asks for dismissal whenever readiness holds a pane with no input box, and the change description states the choice. `SessionBusyError.NoInputBox` and `DismissiblePickerReader` are the minimum the bound and the provider seam need.
- Escalation by `finalize(OutcomeDone)`: it matches the neighbouring "any other re-prompt send failure" exit in the same function, which also finalizes done with the attempts spent, so the stop chain handles it as it handles that exit.
- Every site that sets `unsentReprompt` to false also resets `noBoxReprompts` (`wait.go:347`, `wait.go:582`, `send.go:919`), so the `run.go` field comment is accurate and no stale count survives a new arrival or boundary.
- Commit message and change description claims (once per send, guards, bound of five holds, attempts spent so far) match the diff.
