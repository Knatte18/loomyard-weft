# Round 4 fixer-report — discussion

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  Premise checked against `internal/reedengine/attach.go`: `warnMismatchedClientsLocked` logs a failed `list-clients` at Warn and carries on, so D5's closed list of drops let the selection go out blind.
  D5's Attach bullet now says a failed client listing counts as another client attached and drops the selection at Warn, and the drop list reads "Only a failure to take the lock, read reed state or list the attached clients drops it, logged at Warn."
  The Attach acceptance criterion gains "An attach whose client listing fails leaves the current window unchanged."
- **F2 (LOW), fixed as proposed.**
  D10's fallback parenthetical now reads "(a `top` or `below-parent` strand in window 0, or the session's current window)", matching D3's exclusion of the overview strand.
  `internal/reedengine/strand.go` confirms `top` and `below-parent` are the anchors that place a strand in the strands' window.
- **F3 (NIT), fixed as proposed.**
  The `PATTERN-log-levels` Constraint now cites "D2, D5, D8 and D10"; D9 names no log level.

## Disputed

None.

## Deferred

None.
