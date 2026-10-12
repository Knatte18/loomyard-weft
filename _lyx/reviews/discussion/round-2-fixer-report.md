# Fixer report — discussion round 2

## Changes

- **F1** (MEDIUM, design) — fixed as proposed, option (b).
  D5's Decision now has the same commit amend `pattern/PATTERN-daemon-wakeups.md` so a wait with a deadline of its own and a count cap is outside the rule whatever its length, bounded by that deadline and cap.
  Its rationale states that the background's bounded-wait sentence leaves a 90 s window unplaced until amended, says why per-second captures are acceptable (once per start, stop at ready, tick-capped), and says why a backoff is not taken.
  "At the rule's floor anyway" is dropped from D5 and from the Constraints line, which now cites the amended reading.
  Option (a) was not taken: a backoff that grows while the capture is unchanged leaves a pane that turns ready after a static boot screen unseen for up to the backed-off interval, which undoes D5's goal.
  That reason is recorded as a rejected alternative in `support-log.md`.
- **F2** (MEDIUM, design) — fixed as proposed.
  Premise confirmed: `withPaneTail` ends the unlanded returns in `sendVerified` (`internal/shuttleengine/run.go`), `pasteVerified` and `failUnlanded` and the in-draft and foreign-text readings (`internal/shuttleengine/send.go`).
  D4 now says a busy, in-draft or unlanded send cause already ends with the pane's last lines, and the remaining causes, including a raw key or paste delivery error, report a mechanism failure with no screen, so no cause gets a second tail.
- **F3** (LOW, consistency) — fixed as proposed.
  D4 now says the trigger is never-ready with no cause, names the reed-status retry exits as never-ready with a cause, and keeps the skill-load death as the nil cause that is not never-ready.
- **F4** (MEDIUM, design) — fixed as proposed, first option.
  Premise confirmed: `playInputs` in `internal/shuttleengine/run.go` returns on the first failing `SendKey` after earlier keys were sent.
  D6 now starts the gap on any play that sent at least one key, failed partway or not, leaves it unstarted only for an empty sequence or a play whose first key failed, and states that the play reports whether its first key was sent.

## Disputed

None.

## Deferred

None.
