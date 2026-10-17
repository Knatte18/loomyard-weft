# Fixer-report — darn round 1

## Changes

- F1 (LOW), fixed as proposed: deleted the row `watched blocked stop sends nothing` from `TestDriveNotices` in `internal/sheddriver/drivehandback_test.go`.
  The row `watched awaiting without a notice sends nothing` covers the same input and outcome.
  `lyx gate test ./internal/sheddriver` is green.
  Committed as `672f1e9da`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
