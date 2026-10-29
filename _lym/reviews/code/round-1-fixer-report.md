# Fixer-report, round 1

## Changes

- F1: `ARingAnsweredByAClaimIsNotFollowedByTheBell` in `internal/creelengine/deliver_test.go` now posts its two messages before the start and runs `settle(e, func() { e.start(t) })`; `SetTurn` stays first.
  Fixed as proposed, commit d0df77c1.
  `lym gate test ./internal/creelengine` is green.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
