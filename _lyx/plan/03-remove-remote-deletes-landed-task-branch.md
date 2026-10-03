# Card 3 — remove-remote-deletes-landed-task-branch

**Edit:**
- `internal/fabricengine#branchDirtinessKind`
- `internal/fabricengine#branchDirtiness`
- `internal/fabricengine#checkRemoteBranchRequest`
- `internal/fabricengine#checkBranchDirtiness`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#RemoveResult`

**Create:**
- `internal/fabricengine#dirtyUnlandedRemoteTip`
- `internal/fabricengine/remove_remotetask_integration_test.go`

**Uses:**
- `internal/fabricengine#checkUnlandedWork`
- `internal/fabricengine#deleteRemoteBranch`
- `internal/fabricengine#ownedPairWarpBranch`
- `internal/fabricengine#deleteWarpBranch`

**Intent:**
Implements D8: with `--remote`, after the local task-branch step, `Remove` deletes the pair's task branch (`BranchPrefix + slug`) on the warp repo's origin when its remote tip's work is landed.
This holds on both the full and the finish path.

In `destroy.go`, behind the gate:
- A new branch dirtiness kind, built only through `dirtyUnlandedRemoteTip(parentBranch)`, answers a remote question.
  `checkRemoteBranchRequest` accepts it and requires a non-empty `leaseSHA`, which is the tip under test.
  `checkBranchDirtiness` refuses it for a local delete, the same way it refuses the archived-on-remote kind.
- Its probe applies `checkUnlandedWork`'s rule to the leased remote tip: the tip passes when every commit on it is reachable from another ref, or when merging it into the parent changes nothing (`git merge-tree --write-tree` equals the parent tip's tree).
  For a remote tip, "another ref" excludes every copy of the same branch: `refs/heads/<branch>` and every `refs/remotes/*/<branch>`, through `--exclude` arguments ahead of `--glob=refs/heads/*`, `--remotes` and `--tags`.
  Without the exclusion `refs/remotes/origin/<branch>` always holds the tip, and since the local step already counted that ref, an unlanded pushed branch would lose both copies.
  The parent is the recorded parent branch; empty leaves the reachability check alone.
  `force` is never consulted.

In `remove.go`, a helper observes the remote tip (`ls-remote`), fetches it so its objects are local, and calls `deleteRemoteBranch` with `ownedPairWarpBranch(warpBranch, parentBranch)`, the new dirtiness, and `leaseSHA` set to the observed tip.
A tip that moved after it was observed fails the lease and nothing is deleted.
An absent remote branch, or no origin on the warp repo, is neither deleted nor a reason.
A gate refusal or a failed push is a kept reason, never an error: `Remove` still succeeds.
`RemoveResult` gains `RemoteWarpBranchDeleted bool` (json `remote_warp_branch_deleted`) and `RemoteWarpBranchKeptReason string` (json `remote_warp_branch_kept_reason`, omitempty).
The deletion appends `KindRemoteBranchDeleted` only after it was observed (the executor already does), and adds the task-branch-on-origin step to `Steps`.

Bound: deletes only `BranchPrefix + slug` for the pair being removed, only with `--remote`, only when its work is landed, and only at the leased tip.

Tests in `remove_remotetask_integration_test.go`, against hubforge's bare origin:
- A squash-landed task branch (its tree merged into the parent with a different commit) is deleted on origin.
- An unlanded pushed task branch is kept on origin with a reason naming the unlanded commits, while the local branch is kept too.
- The origin tip moved between observation and deletion (simulated through a test seam on the helper, or by pushing between the two calls) keeps the branch with a lease reason.
- Without `--remote` the origin copy is untouched.

**ImpactSummary:** One new closed-enum dirtiness kind in destroy.go and two new RemoveResult fields; existing local-branch and weft-branch gates are unchanged.

**Commit:** `3: fabric remove --remote deletes a landed task branch on origin`
