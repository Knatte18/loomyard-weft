# Fixer report — merge round 1

## Changes

- **F1 (BLOCKING), fixed as proposed, with one difference in method.**
  `PATTERN.md` is 15666 bytes, down from 16084 and under the parent tip's 15668.
  The merge had kept the longer pre-parent wording on 11 entry lines; I restored the parent's shorter wording on `PATTERN-cwd-resolution`, `PATTERN-sparse-hub-base`, `PATTERN-config-layers`, `PATTERN-fabric-vocabulary`, `PATTERN-mutation-record`, `PATTERN-gogit-read-helper`, `PATTERN-friction-capture`, `PATTERN-review-round`, `PATTERN-testkit`, `PATTERN-hubforge-fixtures` and `PATTERN-documentation-lifecycle`.
  That left the file 10 bytes over, so I also shortened the task's `PATTERN-owner-machine` line ("acting on wake", "verdicts in `ReconcileOrigin`").
  Each dropped clause is still stated in its `pattern/` background file.
  `lyx loom lint-pattern` reports ok.
  Commit: `PATTERN: fix F1`.
- **F2 (LOW), fixed as proposed.**
  `internal/loomcli/ownercheck_test.go` gains the row `OwnerPauseRefusesWithTheResumeWayBeforeTheOwnerVerdict`, run for both `start` and `resume`.
  It covers the run with a live slug-scoped entry whose caller is `pauserecord.OwnerCaller`, and asserts that the refusal carries `pause:` and `"lyx resume`, that the owner seam is called 0 times, and that no bootstrap lock is taken.
  `internal/loomcli/pausehold_test.go` gets `coverRunBy`, the fixture `coverRun` now delegates to.
  `lyx gate test ./internal/loomcli` passes.
  Commit: `loomcli: fix F2`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
