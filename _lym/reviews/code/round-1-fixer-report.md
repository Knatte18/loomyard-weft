## Changes

- F1 (LOW), fixed as proposed: in `TestVerifyGate_RerunsATestFailureOnce` (`internal/landingshed/verifygate_test.go`), the row "a passing verify runs no rerun" gained `log: goFailure`.
  The passed first result now carries a readable test-failure log, so only the Status guard in `rerunTestFailure` keeps it from rerunning.
  Deleting that guard adds a second verify command and fails the row.
  `lym gate test ./internal/landingshed` passed (811 passed, 0 failed).
  Committed as `5f820d57`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
