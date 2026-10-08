# Fixer report — discussion round 3

All eight findings validated against the code and fixed in `_lyx/discussion/decision-record.md`, with the support log updated.

## Fixed

- **F1 (BLOCKING):** confirmed: `hold.go:79` and `wait.go:902` save `run.state` from the `Wait` goroutine outside finalize.
  D7 now names the shared state, gives `Run` a record mutex that guards the mark, `run.state.Outcome` and every save of `run.state`, and has the stop verb set `run.state.Outcome` under it, so a later mid-run save carries `stopped`.
  The guid form is stated as a disk read-modify-write with no in-process `Wait` to race.
- **F2 (MEDIUM):** D3 now says a read at or after the window's close sends no Enter and ends the loop, intervals are cut at the close but the read after an Enter keeps a `submit_redraw_settle_ms` floor, and the overshoot sentence follows from that.
  D6's overshoot list adds the idle wait's last capture and the clear's key choreography.
  The acceptance criterion says the closing confirm read sends no further Enter.
- **F3 (MEDIUM):** the Constraints bullet now has one choosing function return `runOutcomeStopped` unless the handed outcome is `done` or every output file exists; the stop verb and a marked handle's finalize both call it, so the value has one return site and one guarding `allOutputFilesExist` call.
  D7's finalize sentence references that rule.
- **F4 (MEDIUM):** D2 decides that a text that never appeared fails with `ErrSubmissionNotLanded` (support log Q12), so D4 and D6 handle it through their existing sentinel rules.
  D3's clear match covers copies of the text from a doubled replay.
  An acceptance criterion pins the sentinel.
- **F5 (LOW):** D3's clear runs only on an engine with both the idle reading and the clear choreography, a prefix needs at least the needle's length, and the Open risk now covers a pasted placeholder.
  The clear criterion follows.
- **F6 (LOW):** D6 states that the loop captures the pane every tick while a boundary is cleared by an unmatched `turn_start`, that the restoring tick evaluates the gate through `handleGatedBoundary` whether or not an entry is pending, and that the unsent-re-prompt mark is not restored because that evaluation re-derives the re-prompt.
- **F7 (LOW):** the Constraint now says `isTerminalOutcome` is the only change to attach's classification, widens its doc comment, and names the superseded-strand removal as attach's other change.
- **F8 (NIT):** the D1 acceptance criterion now reads "idle on every poll for 10 s".

## Disputed

None.

## Deferred

None.
