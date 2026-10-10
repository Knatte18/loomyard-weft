# Plan-Review round 2 — fixer report

All four findings were checked against the code and fixed.

## Fixed

- **F1 (MEDIUM)**: confirmed.
  Today `blobread.go` maps not-found only on the target lookup and wraps a walk error with `%w`, so folding the walk into the same mapped `readGoGit` call would turn a missing ancestor into `(false, nil)`.
  In card 2's Intent, `HeadContains` no longer counts as a one-closure composite.
  It now calls `readGoGit` twice in sequence: the target lookup, whose not-found folds into false, then the HEAD read and the bounded walk.
  The card states that a not-found during the walk stays a wrapped error, so card 1's `(false, nil)` assertion still tells a bounded walk from an unbounded one.
- **F2 (MEDIUM)**: confirmed.
  `hubforge.AddPairWith` calls `h.Topology.Add` (`internal/hubforge/hub.go`), and card 5 makes Add write the keys.
  Card 5's test now unsets the keys right before the step it checks.
  For Add, they are unset before the pair is created.
  For Remove, they are unset after `AddPairWith` returns and checked absent before `Remove`.
- **F3 (LOW)**: confirmed.
  `TestHistoryReads`'s steps share one repository.
  Card 1 now says the bound step builds its own repository with `newRepo` and leaves the shared fixture untouched.
- **F4 (LOW)**: confirmed against D6.
  Card 5's `PATTERN-hub-store-gc` entry line now carries a short clause saying housekeeping is no destructive primitive under `PATTERN-fabric-destruction-chokepoint`.
  The full carve-out stays in the background file.

## Disputed

None.

## Deferred

None.
