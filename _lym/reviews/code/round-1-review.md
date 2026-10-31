---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/shuttleengine/carryover.go:63
    summary: runTurnStarts now skips a turn start whose Raw line is empty or not found, a change from the old carry-over reading that its doc comment does not state
    fix: In the doc comment of runTurnStarts in carryover.go, add one sentence that a turn start whose Raw line the events file does not hold is not read; operatorInstructions inherits it.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shuttleengine/hold_test.go:336
    summary: No row of the hold-notice table pins the operator-turn skip on a Spec.HoldDeferral run, where the skip must also keep the notice from being queued
    fix: Add a row to TestWait_HeldTurnEndNotice with holdDeferral set and an operator-typed turn (events START, START:what are you doing?, STOP:answered) that expects no notice after the deferral settle and maximum have passed, and NotifiedOffset 0.
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: internal/shuttleengine/doc.go:109
    summary: The new hold-notice paragraph says "read as the carry-over below reads one" and names the sent-texts file, but doc.go has no carry-over or sent-texts paragraph below, so the reference does not resolve
    fix: In doc.go line 109, state the rule without pointing at a section that is absent, for example "a typed instruction past the prompt offset that is not the prompt and that no send into the run typed, which the run's sent-texts record tells", dropping "as the carry-over below reads one".
---

### [LOW] Skipped turn starts without a locatable Raw line are undocumented

**Location:** internal/shuttleengine/carryover.go:63
**Issue:** The old `operatorInstructions` counted every turn start signal. It now reads through `positionedTurnStarts`, which drops a signal whose `Raw` is empty or not found in the data (`wait.go:972-974`). A dropped prompt start would shift which later start counts as the prompt, and a dropped operator start would not be carried over. The Claude engine always sets `Raw`, so production reads the same instructions as before; the `Raw` added to `instructionEngine` in `carryover_test.go` is the only trace of the change.

### [LOW] Operator-turn skip is not pinned on a deferred-hold run

**Location:** internal/shuttleengine/hold_test.go:336
**Issue:** The two new rows use runs without `HoldDeferral`. On a deferred run the new case also has to keep `run.pendingHold` from being set by the arm at `wait.go:327`; a regression there would queue and later send a notice for an operator turn, and no row would fail.

### [LOW] doc.go paragraph points at a carry-over section it does not contain

**Location:** internal/shuttleengine/doc.go:109
**Issue:** The paragraph reads "as the carry-over below reads one" and relies on "the sent-texts file", yet `doc.go` holds no carry-over or sent-texts description (the only matches are this paragraph). A reader of the package doc cannot resolve either, which breaks the self-contained comment rule.

## Rejected

- Held offset on repeated identical turn-end lines (code-correctness item 10): `offsetPastEvent` behaved this way before the diff, so it is not the diff's defect.
- Unlocked copy of `run.state` in `operatorStartedTurn`: `hold.go:133` already passes `run.state` by value on the same path.
- Re-reading the events and sent-texts files at each held end: a cost on a rare branch, not a defect.
- The `>=` boundary in `operatorStartedTurn`: a turn start and the turn end cannot share a line, so the boundary is unreachable and correct.
