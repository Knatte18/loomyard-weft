# Pairs read their branch from the pair, so branch_prefix can change live

Until now, every fabric step after a pair's creation rebuilt the pair's warp branch as `branch_prefix + slug` from the current config.
Setting `branch_prefix` while any pair was live made remove, teardown, the pull request lookup, listing and cleanup look for `<prefix><slug>` and miss the real `<slug>` branches.
A hub with a long-lived beam never has a moment with no live pair, so the prefix could not be introduced at all.

## What changes

- **`Topology.PairTaskBranch`** (new, `internal/fabricengine/pairbranch.go`) reads a pair's task branch from the pair itself, without a network call.
  Its name carries no fabric side, since callers outside fabric's vocabulary owners call it.
  It tries, in order:
  - the branch the task worktree has checked out;
  - the warp branch paired with the branch the records worktree has checked out;
  - the first of the pair's names that a local task or records branch carries.

  A pair's names are `branch_prefix + slug` and the bare slug, which a pair made before the prefix was set carries.
- **Remove** (`RemoveRefusal`, `inspectPair`, `PairParts`) uses the branches read this way.
  When nothing of the pair is left locally, it asks origin under either name.
  `RemoveResult.SiblingBranch` names the records branch it removed, so `lym fabric remove`'s error for a failed origin delete no longer rebuilds the name from the config.
- **The remote teardown job** no longer carries `branch_prefix`.
  It carries the branch names read when the teardown started, because the pair's worktrees are gone by the time the job runs.
- **`DiscardTaskBranch`** discards the branch the pair stands on.
  When no local branch has either name, it asks origin which name is there.
- **`TaskBranchOnOrigin`** answers true for a branch on origin under either name.
- **Branch-to-slug steps accept a branch with the current prefix and one without.**
  These are the cleanup sweep, the origin sweep (`CleanupRemoteWarp` no longer calls a prefix-less branch unmanaged; the weft evidence alone decides), `TaskBranchSlugs`, and `Add`'s check that a `--parent` is on its own pair branch.
- **Records-branch ownership.**
  `ownedManagedBranch` no longer takes a prefix.
  Every caller deletes a records branch, which is managed by its `-weft` suffix, so the prefix test only widened what the gate accepted.
- **Pull request lookup and batten re-adoption.**
  The torn-down pull request lookup in `pairteardown` and batten's re-adoption and incomplete-pair way forward read the task branch through `PairTaskBranch`.
  This read comes after the gone-worktree check, so it falls back to whatever of the pair is left.

`branch_prefix` now only names the branches of pairs created after it changes, and can be set at any time.

## Design decisions

- **Only the pair's own names are read from a worktree.**
  A worktree checked out onto some other branch is not read, so `lym fabric checkout` can never make a teardown delete an unrelated branch.
  The cost: a prefix changed from one non-empty value to another leaves pairs under the first prefix unrecognised.
  The record asks only for the current prefix and none.
- **Origin is asked only when nothing local names the branch.**
  This covers a pair that exists only on origin, so a pair with any local remnant still costs no network call.

## Tests

`TestBranchPrefixChange_KeepsLivePairsManaged` makes a pair under the empty prefix and pushes it, then switches to a topology with `branch_prefix: hanf/`.
It asserts that:

- `PairTaskBranch`, `PairParts` and `TaskBranchOnOrigin` find the old pair;
- a new pair gets `hanf/<slug>`;
- `Remove` deletes the old pair's bare-slug task and records branches.

The existing gate tests drop the prefix argument the test seams no longer take.
