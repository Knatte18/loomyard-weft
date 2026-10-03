# Card 4 — cleanup-remote-task-branch-sweep

**Create:**
- `plan:internal/fabricengine#Topology.CleanupRemoteWarp` -> `func (t *Topology) CleanupRemoteWarp(l *lyxcwd.Location, apply bool, openPRHeads map[string]bool) (RemoteWarpCleanupResult, error)`
- `plan:internal/fabricengine#RemoteWarpCleanupResult` -> `type RemoteWarpCleanupResult struct`
- `internal/fabricengine/cleanupremotewarp_integration_test.go`

**Uses:**
- `internal/fabricengine#dirtyUnlandedRemoteTip`
- `internal/fabricengine#deleteRemoteBranch`
- `internal/fabricengine#ownedPairWarpBranch`
- `internal/fabricengine#Topology.Cleanup`

**Intent:**
Implements the engine half of D9 as a sibling of `Topology.Cleanup`, in a new file `internal/fabricengine/cleanupremotewarp.go`.
`Cleanup`'s signature and its local weft-branch sweep stay unchanged.
`fabriccli` composes the two in card 5, so `fabricengine` gains no import of `internal/githubclient`.

`CleanupRemoteWarp` enumerates every branch on the warp repo's origin (`ls-remote --heads`) and classifies each into one `RemoteWarpBranchEntry` (`Branch`, `Candidate`, `Deleted`, `Reason`, `Error`), returned in `RemoteWarpCleanupResult` (embeds `MutationRecord`; `Entries` always an array; `SkippedReason` when the warp repo has no origin).
A branch is fabric-managed when the weft origin holds an `archive/<slug>/*` tag or a `WeftBranchName(branch)` branch for it, where slug is the branch with `BranchPrefix` trimmed (a branch lacking a non-empty prefix is unmanaged).
The weft origin's heads and tags are listed once per call, not per branch.
A branch is a deletion candidate only when all of these hold:
- it is fabric-managed;
- it is not origin's default branch (`ls-remote --symref origin HEAD`);
- it is not checked out in any hub worktree (`List` over the warp repo);
- it is not in `openPRHeads`;
- its tip's work is landed on origin's default branch, by D8's rule with D8's exclusion of the branch's own copies (`dirtyUnlandedRemoteTip` with the default branch as parent).
Every non-candidate carries a reason naming which of these failed.

Without `apply` nothing is deleted.
With `apply` each candidate goes through `deleteRemoteBranch` with `ownedPairWarpBranch(branch, defaultBranch)`, `dirtyUnlandedRemoteTip(defaultBranch)` and `leaseSHA` set to the tip observed at enumeration.
A gate refusal or a lease failure fills that entry's `Error` and the sweep continues; `Deleted` is set only on an observed deletion.
A nil `openPRHeads` means the caller could not establish the open-PR set, so the method refuses every deletion and returns entries only. Card 5 never calls it that way, and the doc comment says so.

Bound: an unmanaged branch, a live pair's branch, an open PR's head, origin's default branch and any branch with unlanded work are never deleted; there is no `force` parameter, so nothing answers these checks.

Tests in `cleanupremotewarp_integration_test.go` (hubforge bare origins):
- A dry run lists a landed managed leftover as a candidate.
- The same dry run lists, each with its reason: an unmanaged branch, a live pair's branch, a branch in `openPRHeads`, an unlanded managed branch, and the default branch.
- `apply` deletes only the candidate and records exactly one `KindRemoteBranchDeleted`.
- A moved tip keeps the branch with an error.

**Commit:** `4: fabric cleanup engine sweeps leftover task branches on origin`
