# Fixer-report, code round 2

Gate: `lym gate test ./internal/fabricengine` at tier 0 and `--tier 1` both pass after the last change (tier 1: 1719 passed, 0 failed).

## Changes

- **F1** — fixed as proposed. `originreconcile.go` compares `entry.Branch` against `detachedBranch`. Commit `fc85ba03`.
- **F2** — fixed after the parent's decision (creel #692 refused the narrowing).
  `pairTaskBranchLocal` reads the task worktree's branch when it is one of the pair's names or has a records branch in the records repository.
  The records-worktree step reads the warp branch when it is one of the pair's names or exists locally, the same rule seen from the records side.
  New rows `TaskWorktreeOnAnEarlierPrefixBranchIsRead` and `RecordsWorktreeOnAnEarlierPrefixBranchIsRead` pin both steps under an `earlier/` prefix.
  Commit `831ad315`, with self-check follow-ups in the last commit.
- **F3** — fixed with my own change: the proposed `warpBranchFound` field was not made.
  `onOrigin` is non-empty only when no worktree or local branch of the pair remains, and then `found` is false, so the extra field would repeat what `onOrigin != ""` already says.
  `localPartRemains` became `branchPartRemains` (worktrees and local branches, launchers not counted).
  `refuseWhenNothingRemains` asks origin once `branchPartRemains` is false, returns no error for a launcher-only pair when origin holds nothing, and `removePair` adopts `onOrigin` whenever it is non-empty and the mode is not local.
  The test `RemoveDeletesTheBareSlugBranchesOnOrigin` became a table with the launcher-only row `OnlyTheLauncherDirectoryLeft`.
  Commit `df682b7e`.
- **F4** — fixed as proposed, with a different way to make `RecordsRepoRoot` fail.
  New subtest `UnresolvableRecordsRepositoryLeavesTheLocalBranchStep` in `TestPairTaskBranch_ReadsThePairItself`, on a `backend`-anchored hub with the anchor directory deleted from the prime worktree, which makes the worktree list `RecordsRepoRoot` resolves through fail.
  It expects `PairTaskBranch` to return the local branch's name and `PairParts` to complete without error.
  Commit `831ad315`.

## Self-check

- fix-consistency, LOW — `refuseRemoval` doc stale: fixed as proposed.
- fix-consistency, LOW — test file header and test doc describe only the empty prefix: fixed as proposed.
- fix-consistency, LOW — `slugOfBranch` and the branch-to-slug callers know only the current prefix and the bare slug: deferred, see below.
- fix-affected-set, MEDIUM — `addparent.go` reads the parent's branch by name alone: fixed as proposed (same records-branch condition as F2).
- fix-proposal, MEDIUM — records-branch recognition could name another pair's or the prime's branch: deferred, see below.
- fix-proposal, MEDIUM — `RemoveLocal` now asks origin for a launcher-only pair: fixed as proposed in effect. `refuseRemoval` and `refuseWhenNothingRemains` take `localMode`, and a local removal skips the origin ask while a launcher remnant remains.
- fix-code-correctness, MEDIUM — same finding as the one above: merged.
- fix-code-correctness, LOW — the `TaskWorktreeOnAnEarlierPrefixBranchIsRead` row did not isolate the task step: fixed as proposed (the records worktree is removed in that row).
- fix-code-correctness, LOW — no negative row for a worktree on a branch that has a records branch: deferred with the fix-proposal finding above.

All four forks returned a report.

## Disputed

None.

## Deferred

- **Branch-to-slug callers and earlier prefixes** (fix-consistency LOW): `slugOfBranch` and its callers (orphan cleanup, remote warp cleanup, pair listing) accept the current prefix and the bare slug.
  Recognising an earlier prefix there means deleting or listing branches by a prefix no config names any more, which is a choice the decision record does not make, so it goes to the parent.
- **Looseness of records-branch recognition** (fix-proposal MEDIUM, fix-code-correctness LOW): the criterion is the parent's decision in creel #692.
  Narrowing it, for example by also requiring that no other worktree stands on the branch, or adding a negative row for such a branch, changes that decision, so it goes to the parent.
  Git already refuses one branch checked out in two worktrees, and the destructive gates still apply to every teardown.
