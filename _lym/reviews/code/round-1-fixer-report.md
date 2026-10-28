# Fixer-report, round 1

## Changes

- F1 fixed as proposed: `TestHubDobbyDeps_PrimeTidyShuttle` in `internal/beamcli/dobbydeps_integration_test.go` asserts a non-nil `TaskLimit` on the beam's and the prime's deps and never calls the seam.
  Commit `beamcli: fix F1`.
- F2 fixed as proposed: `TestStartReady_TaskLimitHold` in `internal/dobbyengine/starts_test.go` gains a subtest whose seam reads free first and near after, with two ready tasks, and asserts only the first spawns and one hold line is logged.
  Commit `dobbyengine: fix F2`.
- Self-check LOW: the header of `dobbydeps_integration_test.go` now covers the seams filled for both dobbies.
  Commit `beamcli: fix self-check LOW`.

`lym gate test ./internal/dobbyengine` and `lym gate test ./internal/beamcli --tier 1` passed after F1 and F2.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the file header and test name describe only the `PrimeTidyShuttle` seam; fixed as proposed by widening the header, the test name left as the review's `fix` key asks for the assertion in the existing test.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
