# Fixer report — discussion, round 2

## Changes

All edits are in `_lyx/discussion/decision-record.md`.
Each premise was checked against the code: `Finalize.mergeInStep` resolves the local parent branch with no fetch (`internal/landingshed/finalize.go`), `Fabric.Pull` refuses with `ErrWarpDirty` and `ErrMergeInProgress` (`internal/fabricengine/pull.go`), and `FABRIC_SKIP_PUSH` sets `SyncOptions.SkipPush`, which `pushParent` honours through `Deps.PushSkipped`.

- **F1 (BLOCKING), fixed as proposed.**
  D9 now reads two code-side tips, the local parent branch tip and its upstream tip after a fetch, and the parent has moved when either differs from the merged tip; a parent with no upstream or under the push skip reads the local tip alone.
  D9 states that a local parent ahead of or diverged from its upstream at Publish, PR-Gate or Finalize reroutes to Merge-In, where D1's Stuck applies.
  Finalize's retry after merge-in-required uses the same two-ref read.
  The "brings no new code" sentence now says the catch-up merge-in brings no parent code the merge review has not covered, since the local tip is checked equal to the merged tip under the lock.
  Acceptance criteria: a new row for a local parent commit after Merge-In rerouting and sticking; the between-check-and-merge row covers a local commit as well as an upstream push; the Finalize merge-in row is reworded to match.
- **F2 (MEDIUM), fixed as proposed.**
  D1 now runs the fast-forward after Merge-In holds the parent lock, through `fabricengine`'s pull of the parent pair's own worktree, the one Finalize opens, and says why the ordering matters.
  It adds Stuck exits with ways forward for a dirty parent pair, a merge in progress there and no live parent pair, each naming the step in the parent pair and then `lyx loom resume`, and a bound that the fast-forward only moves the parent pair forward to its upstream tip and never touches local commits.
  A matching acceptance criterion is added.
- **F3 (LOW), fixed as proposed.**
  D1 now states that under the told push skip Merge-In skips the fetch and fast-forward and merges and records the local parent tip; D9's local-only read covers the comparison.
  An acceptance criterion for two successive push-skipped landings on one parent is added.

`_lyx/discussion/support-log.md` needed no change.

## Disputed

None.

## Deferred

None.
