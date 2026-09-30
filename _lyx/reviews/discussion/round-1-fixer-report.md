# Fixer report — discussion round 1

All seven findings fixed; `lyx loom validate-discussion` exits 0 on the result.

## Changes

- **F1 (MEDIUM):** D7's first sub-bullet no longer calls `lyx loom start` a re-delivery.
  The reason says the child's driver has not re-stepped yet: `lyx loom start` replaces a driver that has ended, and a live driver is inspected in its pane.
  One rationale line says why start is not a re-delivery.
  The Auto-mode assumption, the battenshed acceptance criterion, support-log Q9 and Interview item 6 now match.
- **F2 (LOW):** D7 states the marker layout: the identity line unchanged, followed by a second line with the decimal length.
  The Mixed binaries risk now covers an older binary reading that marker and accepts the bounded consequence.
  A new acceptance criterion covers the written layout, and support-log Q12 records the decision.
- **F3 (LOW):** The `history_length` open risk became D10.
  Batten's `interrupt_policy` is `""`, so an unchanged read hands back, and waits no longer reset the repair count; both are accepted.
  Q10 is resolved, and Interview item 5 cites the missing `InterruptPolicyFor` wiring.
- **F4 (LOW):** Deleted the Open risks "Activity recomposition" entry and the Notes `persistTransient` line.
  D6 and its acceptance criterion are unchanged.
  Interview item 5 notes that the carried-forward value already matches, so D6 is an explicit rule.
- **F5 (LOW):** The Goal now says the fold covers every budget-exempt wait, including the driver-exit grace wait.
  D1's rationale says only the awaiting wait is unbounded.
- **F6 (NIT):** Dropped the `nowRFC3339` Notes line and the `docs/overview.md` Notes line that repeated Scope's Docs bullet.
- **F7 (NIT):** D7's no-masking rationale now rests on D1 never folding an Awaiting entry.
  The second case reads "any other child history length".

## Deferred

None.
