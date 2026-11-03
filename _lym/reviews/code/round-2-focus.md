---
round: 2
focus:
  - "internal/shuttleengine/send.go `dismissPicker` plays the provider's dismissal key (Escape for Claude, also its interrupt key) through `playInputs`. Settle whether `PATTERN.md` and `pattern/PATTERN-pane-text-injection.md` in the diff name this guarded dismissal as an allowed key delivery, or whether a commit carries a `Pattern-Departure:` trailer naming `PATTERN-pane-text-injection` with a reason."
  - "internal/shuttleengine/send.go `dismissPicker` guards: settle whether the running-turn check (`InterruptGuard.RunningTurn`) and the post-key interrupt settle are exercised by a test through `TestGate_RepromptOverPicker` or `TestDismissPicker`, so deleting either breaks a test."
  - "internal/shuttleengine/wait.go `noBoxReprompts` counting (consecutive no-input-box holds, reset in `Wait` on a new arrival, in `boundaryIdle` in send.go, on a delivered re-prompt, and on a hold of another kind): settle whether each reset is pinned by a test, and whether a dismissal-failed hold in `dismissPicker` still counts toward the bound."
  - "internal/shuttleengine/conditionedsend.go `conditionedReading` sets `SessionBusyError.NoInputBox`: settle whether any code reads it on that path and whether a test asserts it."
  - "internal/shuttleengine/claudeengine/testdata/pane-autocompact-picker.txt: settle whether the fixture's rule glyph matches the glyph `isBoxRule` accepts for other dialog and picker fixtures, so `inputBoxInterior` finding no box comes from the picker shape and not from a glyph no real pane draws."
---

Round 1 left the Escape-dismissal rule conflict, two coverage gaps and three smaller items open; each focus entry names the site to re-check against the current tree.
