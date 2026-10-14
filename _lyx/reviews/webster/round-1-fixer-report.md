# Fixer report, webster round 1

## Changes

- F1: fixed as proposed.
  `planparser.CardCovers` (`internal/planparser/targetdirs.go`) now holds the cover derivation.
  The unexported `cardCovers` in `internal/batcher/boundary.go` and `internal/websterengine/guardstep.go` are deleted, and `renameBoundaries` and `holdsFilesAfter` call `planparser.CardCovers`.
  `lyx gate test` over `internal/planparser`, `internal/batcher` and `internal/websterengine` passes.

## Disputed

None.

## Deferred

None.
