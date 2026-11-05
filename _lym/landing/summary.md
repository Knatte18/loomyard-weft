# fabric: read a pair's branch from its worktree, not from branch_prefix

Setting `branch_prefix` while any pair was live broke that pair's remove, teardown, PR lookup and cleanup.
Each step re-derived the warp branch as `cfg.BranchPrefix + slug` from the current config, so with `hanf/` set it looked for `hanf/<slug>` and missed the real `<slug>` branch.
A hub with a long-lived beam never has a moment with no live pair, so the prefix could not be introduced.

Now a pair's branch is read from the pair itself, and `branch_prefix` only names the branches of pairs created after the change.

Closes #59

## What changes

- New `internal/fabricengine/pairbranch.go` holds the read.
  `Topology.PairTaskBranch` returns the branch the pair's task worktree has checked out, else the warp branch paired with the records worktree's branch, else the first of the pair's names (current prefix, then bare slug) that a local task or records branch carries, else the current prefix's name.
  `PairTaskBranchOrOrigin` also asks origin when nothing local remains.
- `Remove`, `RemoveRefusal`, `DiscardTaskBranch`, the sibling-branch removal, `addparent.go`, `cleanup.go`, `cleanupremotewarp.go`, `pairparts.go` and the PR lookup in `internal/pairteardown/pullrequest.go` take the branch from that read instead of from the config.
- The detached remote teardown and discard jobs no longer carry `BranchPrefix`.
  They carry `WarpBranch` and `WeftBranch`, read when the removal started, because the worktrees are gone by the time the job runs.
- `RemoveResult` gains `SiblingBranch`, the records branch the removal tore down.
- Going from branch to slug (orphan cleanup, remote warp cleanup, pair listing) accepts a branch with the current prefix and one without, through `slugOfBranch`.
- `ownedManagedBranch` and `branchOwnership` no longer take a prefix; ownership is decided by the records-branch scheme (`WeftWarpSlug`), so a pair made under an earlier prefix stays managed.
- The `branch_prefix` comment in `template.yaml` and the fabricengine `doc.go` describe the new meaning.

## Design decisions

- The worktree is the source of truth, with no new record and no config lookup.
  The warp worktree's path follows the slug and git reports its checked-out branch, so there is nothing to keep in sync.
- A task worktree's checked-out branch is accepted when it is one of the pair's names or has a records branch in the records repository.
  A pair made under any earlier prefix, empty or not, therefore stays recognised after `branch_prefix` changes again.
  A worktree on any other branch is ignored, so a pair checked out onto an unrelated branch never hands that branch to a teardown.
- Origin is asked only when no worktree and no local branch of the pair is left.
  A pair whose only local remnant is its launcher directory is asked too, since the directory names no branch, except in `Remove`'s local mode, which makes no origin call for a pair that still has a remnant.
- When both a bare-slug and a prefixed branch of one slug exist, the prefixed name wins in the local-branch fallback.

## To check by hand

- Not run: a live end-to-end with pairs created under `branch_prefix: ""`, then `branch_prefix: hanf/` set, then remove, teardown, PR lookup, listing and cleanup of those pairs, and a new pair that gets `hanf/<slug>`.
- `internal/fabricengine/remove.go`, `internal/fabricengine/taskbranchdiscard.go`, `internal/fabricengine/destroy.go`: a detached-HEAD literal may still sit next to the `detachedBranch` constant (finding "detached-literal-survives-next-to-constant", blocking when raised).
  Confirm no remaining comparison uses the literal.
- `internal/fabricengine/pairbranch.go`: a prefix changed between two non-empty values (finding "prefix-changed-between-nonempty-values-unrecognised").
  Confirm a pair made under `a/` is still recognised after the prefix becomes `b/`.
- `internal/fabricengine/pairbranch.go`: an unresolvable records repository skips the records-worktree step and falls through to the local-branch step (finding "unresolvable-records-root-skip-untested").
  Confirm the pinning test in `internal/fabricengine/pairbranch_integration_test.go` covers that skip.
