# Fixer-report, round 1

## Changes

- F1 (LOW, fixed as proposed, plus the optional part): `startReady` in `internal/dobbyengine/starts.go` holds a refused task only when it is also not owned elsewhere, has no pending dependencies and is not covered by a pause, so a task another dobby or condition owns is skipped silently as before.
  The `heldStart` doc comment and the `doc.go` bullet state those conditions.
- F2 (LOW, fixed as proposed): the `lym beam status` help text in `internal/beamcli/cli.go` lists a start batten refuses among the waits and says a ready task shows its last refused start.
- F3 (LOW, fixed as proposed): a `TestBuildView` row, `ATaskWithAStartRefusalTakesNoMaxRunsSlot`, with `MaxRuns` 1 and a refused task listed before a ready one, expects the ready one to carry no `AtMaxRuns`.

The gate for `./internal/dobbyengine` and `./internal/beamcli` passed (tier0).
No test row pins the new F1 conditions; the review asked for none.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, `doc.go` bullet and `startReady`/`heldStart` comments omit the new conditions — fixed as proposed for `doc.go` and `heldStart`; `startReady`'s comment does not describe the hold and is left unchanged.
- fix-proposal: LOW, same `doc.go` gap — fixed as proposed (duplicate).
- fix-affected-set: LOW, same `doc.go` gap — fixed as proposed (duplicate).

## Disputed

None.

## Deferred

None.
