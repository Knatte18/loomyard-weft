# Shuttle closes Claude's autocompact picker and bounds a held re-prompt

When Claude Code shows its "We recommend compacting earlier" autocompact picker in a shuttle session, the picker takes the input box's place.
A gate re-prompt into that pane was held as "no input box is on screen", spent no attempt and was retried every tick with no bound, so the run waited until its deadline.
This change closes the picker the way an operator would, and bounds the wait for a picker shuttle does not recognise.

## What changes

- **New optional engine capability.**
  `shuttleengine.DismissiblePickerReader` lets a provider recognise a known mid-session picker that covers its input box and name the keys that close it while keeping the current setting.
- **Claude recognises the autocompact picker.**
  `claudeengine`'s `DismissiblePicker` matches a small table of pickers by wording, today the autocompact picker ("We recommend compacting earlier" together with "esc not now").
  It also requires that no input box is on screen, so a transcript quoting the wording above a live box does not match.
  It answers one Escape, the "not now" choice.
- **The verified send dismisses it.**
  During a send's idle wait, a reading that is not ready over a recognised picker gets the picker's keys at most once per send.
  The keys go out only when the operator was not active in the pane within the quiet window and the engine's `InterruptGuard` reads no running turn, since Escape is also Claude's interrupt key.
  The pane then settles for the engine's interrupt settle and is read again, so the send goes on once the box is back.
- **A missing input box no longer holds a re-prompt forever.**
  `SessionBusyError` gains `NoInputBox`, set when the hold is a pane showing no input box.
  The gated wait loop counts consecutive re-prompt sends held that way.
  At `maxNoInputBoxReprompts` (5) it ends the loop with the attempts spent so far, with a Warn line naming the hold, the same way an unexpected re-prompt send failure already ends it.
  Any other kind of hold, a delivered re-prompt, a new arrival or a writer turn starting resets the count.
  Each held send has already waited out its idle window (60 s by default), so an unrecognised picker ends the wait after a few minutes instead of at the run deadline.

## Design decisions

- **Dismissal lives in the send's idle wait, not in the gate loop.**
  Every verified send goes through that wait, so creel mail and continue lines into a pane behind the picker recover too, not only gate re-prompts.
  The read-only idle probe (`Runner.SessionIdle`) and the conditioned send, which refuses at once, press nothing.
- **Escalation reuses the existing end of the loop.**
  Finalizing done with the attempts spent hands the failed gate to the caller's stop chain, which already handles a re-prompt that could not be sent, so no new outcome or notice path was added.
- **An operator in the pane is never overridden.**
  An active operator holds the send with no key, and that hold is not a no-input-box hold, so it does not count toward the bound either.
- **No Claude Code setting is changed**, as the board entry records.
- **Pending gate sends are not bounded.**
  A `PassOnCap` entry's pending send already never ends the loop by design; it still benefits from the dismissal.

## Tests

- `claudeengine`'s `TestDismissiblePicker` pins the picker reading against a pane fixture and the near-misses: the wording quoted above a live box, one phrase alone, the rewind picker and an idle box.
  The fixture `testdata/pane-autocompact-picker.txt` is drawn from the board entry's description of the picker, not transcribed from a live pane.
- `shuttleengine`'s `TestGate_RepromptOverPicker` runs a gated wait over a pane fake that can show a picker.
  A known picker is dismissed and the re-prompt delivered.
  A known picker whose key does not close it, and an unknown one, end the loop after the bound with no attempt spent, one Escape per held send for the known one and none for the unknown one.
  An active operator gets no key and holds until the deadline.
