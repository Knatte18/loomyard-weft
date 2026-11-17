# Round 2 fixer-report — plan

## Changes

- **F1 (BLOCKING, scope): fixed as proposed.**
  The premise holds: `r.pool.TryAcquire` at `internal/gatecli/testmapaudit.go:52` is `Pool.TryAcquire`'s one non-test caller,
  and `index.ProfiledFuncs` at `internal/gatecli/testmappackage.go:475` is `ProfiledFuncs`'s one.
  Card 1's **Delete:** list gains `internal/gateslot#Pool.TryAcquire`, `internal/gateslot#TestTryAcquire_TakesAFreeSlotAndNeverWaits` and `internal/testcover#FuncIndex.ProfiledFuncs`.
  Card 1's Intent says why each goes, its Docs paragraph drops the `Pool.TryAcquire` sentence from `internal/gateslot/doc.go`, and its ImpactSummary names both deletes.
  Card 2's Intent drops the `ProfiledFuncs` sentence from `internal/testcover/doc.go`.
- **F2 (MEDIUM, design): fixed differently in its test row.**
  The premise holds: `diffBaseGroups` takes a tier's base from `green.record.newestWholeAncestor` whenever `green` is non-nil (`internal/gatecli/scope.go:393-394`).
  As proposed, card 1 now passes a nil green record to `computeScope`, drops `internal/gatecli#openGreen` from **Uses:**, and states the merge-base and whole-module outcomes.
  The sentence about the green record now says the map run reads no green record at all.
  "Picked once from tier0's base" now reads "picked once from the merge base or the `--base` commit".
  I did not add the proposed row to `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps`.
  That fixture's module lies under `backend`, so `impactset.DeriveTargets` returns whole whatever the base is (`internal/impactset/targets.go:54-55`, "no go.mod at the worktree root"), and the row would pass with the bug in place.
  The new subtest `AComputedScopeTakesNoBaseFromTheGreenRecord` goes in `TestGateTestMap` instead, on the prime, which has a root `go.mod` and no parent branch.
  The subtest sets tier0's command to `true`, records tier0 whole-green at HEAD with `--wait --tier 0 --all`, and wants `test --wait --map` with no scope flag to build every package `--all` builds.
  With the bug, the base is HEAD, the diff is empty, and the call builds nothing.
  The subtest removes the green record and restores tier0's command when it ends, so card 3's `AMapRunWritesNoRunRecordAndAGreenRecordDoesNotStandInForIt` still starts with no green record.

`lym loom validate-plan` reports `ok: true` after every change, the self-check's included.

## Self-check

- fix-consistency, LOW: the new subtest's exit code was unstated, though card 3 makes a map run over the `--all` set exit 1. Fixed as proposed: the subtest asserts the built set alone, never the exit code.
- fix-proposal, LOW: `FuncIndex.funcsHolding`'s `coveredOnly` parameter is always true once `ProfiledFuncs` goes. Fixed as proposed:
  card 1 re-signs `internal/testcover#FuncIndex.funcsHolding` without the parameter and lists `internal/testcover#FuncIndex.CoveredFuncs`, the caller the plan gate requires, and its Intent says why.
- fix-claims: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
