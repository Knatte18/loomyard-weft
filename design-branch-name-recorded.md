# A pair reads its branch from its worktree, so branch_prefix can change with pairs live

`branch-name-recorded` · Task · bug, fabric · running · Describe

## Seen

The hub NORCE Models wants its LoomYard branches named `hanf/<slug>`, through fabric's `branch_prefix` (currently `""`, template).
The prefix cannot be set safely while any pair is live: the code re-derives the warp branch as `cfg.BranchPrefix + slug` from the current config at every later step instead of reading the branch the pair was created with:

- `internal/fabricengine/remove.go` (`warpBranch := t.cfg.BranchPrefix + slug`, the local branch check, the weft sibling via `RecordsBranchName(t.cfg.BranchPrefix+slug)`).
- `internal/fabricengine/pairparts.go` (`remoteHeadTip` on `BranchPrefix+slug`; slug recovered with `CutPrefix(branch, cfg.BranchPrefix)`).
- `internal/pairteardown/pullrequest.go` (`taskBranch = cfg.BranchPrefix + slug` when finding the PR).
- `internal/fabricengine/remoteteardown.go` (the job carries the current `BranchPrefix`).
- Also `taskbranchdiscard.go`, `cleanup.go`, `cleanupremotewarp.go`, `addparent.go`, and the managed-branch ownership checks in `checkout.go` and `reconcile.go`.

So setting the prefix while beams and tasks run makes their teardown, PR lookup and remove look for `hanf/<slug>` and miss the real `<slug>` branches.
A hub with a long-lived beam never has a moment with no live pair, so the prefix effectively cannot be introduced.

## Chosen fix (operator, 2026-11-05)

The pair's worktree is the source of its branch name, not a new record and not the current config:

- The warp worktree's path follows the slug, and git knows which branch each worktree has checked out (`git worktree list`), so every step that starts while the pair's worktree exists reads the warp branch from it, and the weft branch from the weft worktree the same way.
- Steps that run after the worktree is gone, the detached remote teardown and discard jobs, carry the branch names read when the removal started; the job already has fields for them, so it stops carrying `BranchPrefix` for that purpose.
- The PR lookup in `pairteardown` takes the branch read from the worktree.
- The steps that go the other way, from a branch to its slug (orphan cleanup, remote warp cleanup, pair listing, managed-branch ownership), accept a branch with the current prefix and one without, so pairs created before the prefix changed stay managed.
- `branch_prefix` then only names the branches of pairs created after the change, and can be set at any time.

## Acceptance

- With live pairs created under prefix `""`, setting `branch_prefix: hanf/` leaves their remove, teardown, PR lookup, listing and cleanup working, and new pairs get `hanf/<slug>`.
