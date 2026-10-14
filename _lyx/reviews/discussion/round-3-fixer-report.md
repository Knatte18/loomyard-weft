# Round 3 fixer-report

## Changes

- F1, fixed as proposed: in `decision-record.md` D7, replaced the false gating-class premise with the soft-cap account: a MEDIUM `decision`-class entry never makes the judge rule CONTINUE, but in the soft-cap window it counts as design-wrong unless tagged `mechanical` (`designWrongKeys` in `internal/shedadapters/bouncerfiles.go`), so it can block an automatic accept and escalate with cause circling to the parent.
  Narrowed the Acceptance criteria line to a round whose only open entry at MEDIUM or worse is such a finding, which the judge rules CONVERGED.
- F2, fixed as proposed: in `support-log.md` Interview item 6, the "scenarios MEDIUM + pattern-fit LOW" bullet now notes the later reversal to D5's committed-copy check, and the "board-fit + leanness LOW" bullet now says the CLAUDE.md Constraint was dropped.

## Disputed

None.

## Deferred

None.
