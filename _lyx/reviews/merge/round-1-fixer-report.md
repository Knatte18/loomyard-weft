# Fixer-report, merge round 1

## Changes

- F1 (fixed as proposed): `wire` in `internal/loomcli/wiring.go` sets `websterGeom.VerifyHold = verifytree.NewPauseHold(pauseGeom, nil)`; `pauseGeom` was already read above, so no reordering was needed.
  The `WebsterDeps.Geom.VerifyHold` nil-legal entry is gone from `internal/testkit/envkit/envkit.go`.
  `shedfake.WebsterSeams` now fills the field, and `internal/battencli/wire_seams_test.go` lists it as a loom-only nil seam.
  Commits: `loomcli: fix F1` and `battencli: fix F1`.
- F2 (fixed as proposed, except the nil-hold row): `TestVerifyGate_TellsTheGeometryHoldOnTheFirstRunAndTheRerun` in `internal/websterengine/verifygate_integration_test.go` counts `Wait` calls of a told hold across a failing first run and a passing rerun, and asserts two.
  The nil-hold row was not added: every other `NewVerifyGate` test there already runs with a nil hold.
  Commit: `websterengine: fix F2`.

Tier 1 passes over `websterengine`, `loomcli`, `testkit`, `shedrecipe` and `battencli`.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
- fix-consistency: MEDIUM, `battencli` seam test lacked the `WebsterDeps.Geom.VerifyHold` nil-legal entry; fixed as proposed.

## Disputed

None.

## Deferred

None.
