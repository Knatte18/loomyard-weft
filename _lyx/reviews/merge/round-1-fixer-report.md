# Fixer-report — merge round 1

## Changes

- F1 (LOW, scope): fixed as proposed, commit `07c15a65`.
  `internal/hubreconcile/ensure_integration_test.go` gained a `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` row with `staleRun{boardBehind: true, unmigratedBoard: true}`.
  The board holds the pre-change base files and no marker, upstream moves on another path, and Ensure replays the strip commit over it.
  The shared assertions check the pulled upstream commit under the board commit, the stripped base files, the marker in the walk's commit and the written stamp.
  `lyx gate test --tags integration ./internal/hubreconcile` is green.

- Gate finding (PATTERN headroom): `PATTERN.md` had grown 165 bytes past its headroom line, so no test ran.
  Seven overview entries lost wording their background files already carry, with no background file edited: `PATTERN-sparse-hub-base`, `PATTERN-review-round`, `PATTERN-friction-capture`, `PATTERN-cwd-resolution`, `PATTERN-config-layers`, `PATTERN-hubforge-fixtures` and `PATTERN-testkit`.
  The overview is back to 15736 bytes and `lyx loom lint-pattern` reports ok.
  The tagged gate run has not been repeated since this cut.

## Disputed

None.

## Deferred

None.

## Self-check

All four forks returned.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
