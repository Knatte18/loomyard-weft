# Fixer report — plan, round 2

## Changes

- F1 (LOW, scope), fixed as proposed: in `_lyx/plan/05-shuttleengines-registry.md`, the `TestNames_EqualModelspecEngineNames` bullet now overwrites the first element of a returned `modelspec.EngineNames()` slice and asserts a fresh call still equals `Names()`, pinning the copy contract.
  The card states that sort order cannot be pinned while `claude` is the only registered name.
  No top-level test is added, so the `internal/shuttleengines: 3` budget row stands; no target list or verify command changed.

## Disputed

None.

## Deferred

None.
