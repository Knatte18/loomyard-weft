# Fixer report — plan, round 3

I checked both findings against the plan and the code.
Both premises held, so both are fixed.
`lyx loom validate-plan` passes after the fixes.
Its one informational finding, `premerge_test.go` as a new unit, predates this round.

## Fixed

- **F1** (an unreadable strand table becomes a Stuck reason with an empty guid): confirmed.
  Card 5's `conflictSessionStopper` returns a status error with no guid, and `preMerge.clear` mapped every stop error to the guid-naming Stuck reason.
  Card 5 now says:
  - The `StopConflictSession` contract returns an empty guid beside an error when the strand table could not be read.
  - `clear` returns an empty-guid stop error as an error, which `gateStop` maps the same way as a probe error.
    It leaves the merge untouched and does not abort.
  - The guid-naming Stuck reason is kept for an error that carries a guid.
  - The planned `premerge_test.go` table gains a row: the stop seam returns an empty guid and an error, and the row asserts an error return, no `AbortMerge` call and no merge-in.
  - `TestConflictSessionStopper`'s status-error row asserts the empty guid.

  No target or top-level test was added.
- **F2** (the parent-branch read can replace another row's `merge_in_progress` refusal): confirmed.
  `refuseOverUnfinishedMerge` at `internal/loomcli/start.go:141` sets the refusal's kind and `conflicts` list only on the non-error path.
  So a read placed before the producer and verb checks would turn those refusals into a plain error.
  Card 6 now has the guard read the parent branch only for a parked `merge-in` whose producer is Publish or Finalize.
  Every other parked state takes today's refusal without the read.
  Two test changes pin that order:
  - The `ProbeErrorRefuses` row sets the current producer to Publish.
  - The `SpawnRefusals` rows for another producer and for the `merge` verb run with a failing `recordedParentBranch` seam, and still expect `merge_in_progress`.

  No target or top-level test was added.

## Disputed

None.

## Deferred

None.
