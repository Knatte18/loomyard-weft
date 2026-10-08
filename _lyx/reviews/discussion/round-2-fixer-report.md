# Fixer report — discussion round 2

Every finding was validated against the code and fixed.

## Fixed

- **F1 (BLOCKING), leftover text blocks D6's retry.**
  Confirmed: `confirmSubmitted` in `run.go` returns with the text still in the box, and D1 requires an empty box.
  D3 now clears the send's own text before returning `ErrSubmissionNotLanded`, through an engine-owned choreography sent only when a reading shows the box holding the sent text, a prefix of it or a paste placeholder, never with a key that interrupts a running turn, and re-read afterwards.
  D4's late-landing check moved onto D3's pre-clear reading and a signal read, since the box is empty after the clear.
  Acceptance criteria for the clear and for the re-sent re-prompt; Open risks names the operator-draft residual; Q11, an auto-mode bullet and a rejected alternative recorded.
- **F2 (MEDIUM), no override at the gate boundary.**
  D6 applies D1's interrupt and 10 s override and restores a boundary cleared by an unmatched `turn_start`; the Bound, the D6 criterion and the Open risks bullet now agree with D1.
- **F3 (MEDIUM), undefined appearance check.**
  D2 names `sendVerified`'s appearance check and replay and puts both inside D3's window.
  D3 guarantees one confirm read after every Enter.
  D6 takes one deadline budget at the send's start and names the overshoot as the tail past the window.
  Criteria added for the post-Enter read and the deadline budget.
- **F4 (MEDIUM), burler's route to the handle form.**
  Confirmed: `burlerengine.Handle` has no stop, and `stopHalf` stops by guid while the half's `Wait` runs in `join`.
  D7 adds the stop method to the `Handle` seam and routes `stopHalf` through it; webster and attach keep the guid form.
  `Run` gains a mutex guarding the mark, held by the stop verb and by finalize, and the mark is set before the strand is removed.
  Burler criterion added for a half stopped during `join`.
- **F5 (MEDIUM), tripwire blind to the `stopped` write.**
  Constraint now names `runOutcomeStopped`, returned from one function so the scan sees it, added to `negativeVerdictMarkers`, with its return and `allOutputFilesExist` call pinned, and `stopped` added to the pattern background file's and `wait.go`'s lists of negative answers.
- **F6 (LOW), Constraint contradicted Scope.**
  Reworded: the file-contract-first branch is unchanged, and `isTerminalOutcome` is attach's only change.
- **F7 (LOW), `TurnStartInterrupt` misplaced.**
  D1 names `SessionProber.TurnStartInterrupt`, says a parser-only engine has the 10 s path alone, and counts `turn_end` and `api_error_turn_end` as turn ends; the D1 criterion says so.
- **F8 (LOW), D8 unscoped.**
  The invariant text is scoped to Go outside `internal/shuttleengine`, and names finalize's cleanup and `abandonStartup` as shuttle's own teardowns.

The support log's Review rounds section records this round.

## Disputed

None.

## Deferred

None.
