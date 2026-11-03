# shuttleengine: dismiss the autocompact picker, bound no-box re-prompts

A gate re-prompt no longer stalls when Claude Code's "We recommend compacting earlier" autocompact picker covers a shuttle session's input box.
Before, the send was held as "no input box is on screen; a menu or picker may be open".
A held re-prompt spends no attempt and never escalates, so the run waited until its deadline for an operator to press Escape.

## What changes

- A verified send's idle wait now closes a known mid-session picker.
  `claudeengine` implements the new optional `shuttleengine.DismissiblePickerReader`, and `Claude.DismissiblePicker` matches the autocompact picker by its wording ("We recommend compacting earlier" and "esc not now").
  It answers one `Escape`, which keeps the current setting.
  The idle wait plays the key, settles for the engine's interrupt settle and reads the pane again, so the send goes on once the input box is back.
- A pane that still shows no input box ends the wait.
  `SessionBusyError` gains `NoInputBox`.
  After `maxNoInputBoxReprompts` consecutive re-prompt sends held that way, the gated loop finalizes done with the attempts spent so far, where it used to retry until the run deadline.
  The counter resets on a successful re-prompt, a new arrival, a started writer turn and a held send of any other kind.
- `PATTERN.md` and `pattern/PATTERN-pane-text-injection.md` name the dismissal as the second exception to the guarded-interrupt rule.

Closes nothing on the issue tracker; the board entry lists no issue.

## Design decisions

- The dismissal fires at most once per send attempt, and never selects an entry.
- Guards: it fires only when the engine's `InterruptGuard` reads no running turn, and only when no operator was active in the pane within `operator_quiet_s`.
  An active operator holds the send without a key, because the operator may be answering the picker.
- Recognition needs every needle on screen and no input box, so a transcript quoting the wording above a live box is not the picker.
  A release that rewords the picker leaves it unrecognised; the bounded escalation then ends the wait.
- A dismissal key that cannot be sent holds the send as a no-input-box hold, so it counts toward the bound.
- No Claude Code setting suppresses the picker, and the picker is rare, so shuttle handles it rather than a settings change.

## To check by hand

- `dismiss-picker-escape-outside-pane-text-injection` (BLOCKING, design): the dismissal's `Escape` goes out through `playInputs` in `internal/shuttleengine/send.go`, outside reed's guarded paste-submit.
  Check it against `PATTERN-pane-text-injection`, which now names it as an exception.
- `dismiss-picker-running-turn-guard-untested` (MEDIUM, scope): check that a test pins the running-turn guard in `dismissPicker`.
- `no-box-reprompts-reset-semantics-untested` (MEDIUM, scope): check that a test pins when `Run.noBoxReprompts` resets.
- Not run: the dismissal against a live Claude Code session showing the real picker; tests use a captured fixture, `pane-autocompact-picker.txt`.
