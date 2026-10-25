## Changes

All four findings fixed as proposed, one commit per finding (F1 and F3 share `passes.go`, so they share a commit).

- F1: `planPass` returns a full pass (`passFullNoReopen` while the origin-owed marker stands) in the uncovered tail when `trigger.Lift` is set.
  The `CoveredLiftGivesFull` row is replaced by `LiftGivesFull` and `LiftWithOriginOwedGivesFullWithNoReopen` (covered=false), and the lift-while-in-flight test asserts the lift's pass is full.
  The `planPass` doc comment and the `doc.go` continue-pass bullet name the lift.
- F2: `boardCriticalSection` runs `migrateDraftStatus` right after the load, before the `loaded` clone and `fn`.
  `TestDraftMigration` gains a row where a null-status release is the first write to a board without `draft_status` and the task stays released.
- F3: `drivePasses` deletes `liftPending[slug]` when the run is not running or carries a cancel marker and no pass is in flight, and for slugs no longer listed.
  A test row covers a lift seen while the run is blocked.
- F4: `ChildArbiterStop.On` and `Timeout` comments name the arbiter keys in `shed.yaml` with the recipe's module override.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: `doc.go` continue-pass bullet lacked the lift condition. Fixed as proposed.
- fix-proposal, LOW: the `liftPending` delete ignored `InFlight`, against the review's proviso. Fixed as proposed (`!trigger.InFlight` guard).
- fix-proposal, LOW: no row pinned the covered-and-lift branch. Fixed as proposed (`CoveredLiftGivesFull` row restored beside the new rows).

Gate: `lyx gate test` green for `internal/dobbyengine`, `internal/boardengine` and `internal/battenshed`.

## Disputed

None.

## Deferred

None.
