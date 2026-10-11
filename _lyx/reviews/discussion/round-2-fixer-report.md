# Fixer report: discussion, round 2

## Changes

- **F1** (BLOCKING, fixed as proposed): premise confirmed: `readRunSessionState` folds signals from the run record's `PromptOffset` (`internal/shuttleengine/sessionread.go`), reed relaunches with the stored `ResumeCmd` (`internal/reedengine/strand.go`), and `sessionEndReasonEndsProcess` reads `logout`, `prompt_input_exit` and `other` as process-ending, so an earlier launch's `SessionEnd` is the newest signal until `SessionStart(resume)` lands.
  D7 now counts a process-ending `SessionEnd` only when its line lies past the events-file size this wait took at its first liveness tick, or this wait read its session id alive.
  The bound restates the resumed-launch guarantee for both paths, and a wait attached after the death now waits out its deadline as today.
  A Death criterion covers an earlier launch's `SessionEnd` as the newest signal with the resumed process not yet registered.
  D1 already records hook lines for a killed process, so it needed no change.
- **F2** (MEDIUM, fixed): premise confirmed against `confirmPasted` in `internal/shuttleengine/send.go`: a box holding other text beside the sent text fails with `ErrSentTextInDraft` and sends no key.
  D8 now decides that the post-paste confirm keeps its exact-text reading, with no residue reading: such a box fails the send with `ErrSentTextInDraft`, quoting the box, sending no key and recording the pending send, and holds later sends until the operator clears it.
  It also states that a surviving real character is submitted by the paste-submit's own Enter.
  The Stray character criterion gains the case.
- **F3** (LOW, fixed as proposed): the `PATTERN-pane-text-injection` Constraints entry now amends the lead sentence's merge, submit and wipe clauses with D8's bounds, which are the guard reading, `operator_quiet_s` and one clear per send attempt.
  Its `PATTERN.md` sentence now says the residue reading applies at the guard reading only, and that the confirm keeps exact-text, so the line stays unchanged in line with F2.
- **F4** (LOW, fixed as proposed): the Turn 8 ledger line now records the alive-first rule D7 states.
- **F5** (NIT, fixed as proposed): the treat-as-empty rejection now gives the reason D8 differs: the clear makes the delivered text exact in every case but a real character surviving the clear and the settle.

## Disputed

None.

## Deferred

None.
