# Fixer-report, round 1

## Changes

- F1 fixed as proposed: the `LogFileName`, `Live` and `SpawnDetached` docs in `internal/dobbyengine/start.go` no longer name a marker directory.
- F2 fixed as proposed: the `DobbyGeometry` doc in `internal/hubgeom/dobbygeom.go` names the lock, state and log paths and the tidy directory.
- F3 fixed as proposed: the spawn row of `TestCountLandingsAndTidy` asserts `Tidy.ReportPath` equals `TidyDir` joined with the UTC timestamp name.
  No new test.

Changes from the self-check, below: the `TestDobbyGeometry` doc in `internal/hubgeom/dobbygeom_test.go` takes the same F2 wording, and the spawn row's `Tidy` state check uses `t.Fatalf`.

## Self-check

- fix-code-correctness, LOW: the new assertion dereferences a nil `Tidy` after an `Errorf` check. Fixed with my own change: that check is now `t.Fatalf`.
- fix-proposal, LOW: same nil dereference as above. Merged with it and fixed the same way.
- fix-affected-set, LOW: `TestDobbyGeometry` doc in `internal/hubgeom/dobbygeom_test.go:60` still said marker paths, the twin of F2. Fixed as proposed.
- fix-consistency: no findings.

## Disputed

None.

## Deferred

None.
