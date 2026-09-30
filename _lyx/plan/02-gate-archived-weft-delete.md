# Card 2 — gate-archived-weft-delete

**Edit:**
- `internal/fabricengine/destroy.go`
- `internal/fabricengine/destroy_test.go`
- `internal/fabricengine/export_test.go`
- `internal/fabricengine/destroyremote_integration_test.go`

**Create:**
- `internal/fabricengine#ownedPairWeftBranch`
- `internal/fabricengine#dirtyArchivedOnRemote`

**Uses:**
- `internal/gitrepo#Repo.DeleteRemoteBranchLeased`
- `internal/fabricengine#WeftBranchName`
- `internal/fabricengine#WeftWarpSlug`
- `internal/fabricengine#primaryWeftBranch`

**Intent:**
Extend the gated remote-branch deletion in `destroy.go` so Add can replace a provably-archived leftover of its own weft branch, per the Fabric Destruction Chokepoint Invariant.
Three additions, each keeping the closed enums closed:

- **Ownership kind `branchOwnershipPairWeft`**, constructed only by `ownedPairWeftBranch(l, warpBranch)`.
  `resolveBranchOwnership` dispatches it to a new predicate: the branch must be exactly `WeftBranchName(warpBranch)` and accepted by `WeftWarpSlug` (both pure name checks, run first so a mismatched name refuses without spawning git), then must not be `primaryWeftBranch(l)`, failing closed when the primary cannot be read.
  It deliberately has no checked-out test: at Add's step 12 the same-named local branch is the replacement, checked out at the new weft worktree, which is exactly what `ownedManagedBranch` refuses.
- **Dirtiness kind `branchDirtinessArchivedOnRemote`**, constructed only by `dirtyArchivedOnRemote(archiveTag)`, where `archiveTag` is the `archive/<slug>/*` tag the pre-flight proved covers the remote tip.
  `branchDirtiness` gains an `archiveTag` field serving this kind only.
  `checkRemoteBranchRequest` answers it without the checked-out probe: an empty `archiveTag` refuses on `CheckDirtiness` (the tip is covered by no archive tag), and so does an empty lease SHA, since the reachability proof holds only for the SHA it was computed against.
  `checkBranchRequest` refuses this kind on `CheckDirtiness`, since it answers a remote question and a local `git branch -D` has no remote tip to prove.
- **Lease SHA:** `remoteBranchRequest` gains a `leaseSHA` field, documented as the remote tip the caller observed.
  `deleteRemoteBranch` calls `DeleteRemoteBranchLeased` when `leaseSHA` is non-empty, recording `KindRemoteBranchDeleted` via `AppendRef` only on success; an empty `leaseSHA` keeps today's `DeleteRemoteBranch` path, so the two existing constructions (`cleanup.go`, `weftwiring.go`) are unchanged.

Tests:
- `destroy_test.go` (hermetic, alongside the existing `RemoteBranch*` subtests of `TestGate_CheckOrdering`): an unset ownership with the new dirtiness refused on ownership; an unset dirtiness with the new ownership refused on dirtiness; a branch other than `WeftBranchName(warpBranch)` refused on ownership; a `branchRequest` declaring the new dirtiness kind refused on dirtiness.
- `export_test.go`: a seam `DeleteArchivedWeftBranchForTest(l, repoDir, warpBranch, branch, archiveTag, leaseSHA string) (Mutations, error)` that builds the request from the two new constructors and drives `deleteRemoteBranch` with a fresh recorder, returning its snapshot.
- `destroyremote_integration_test.go`, on a `hubforge` hub with a pushed pair: the pair's weft branch, still checked out at its weft worktree, is deleted from origin with a valid tag and a lease at the remote tip, and the record holds one `remote_branch_deleted` entry; the primary weft branch is refused on ownership; an empty archive tag is refused on dirtiness and origin keeps the branch; a valid archive tag with an empty lease SHA is refused on dirtiness and origin keeps the branch; a stale lease (origin's weft branch advanced from a second clone of the weft bare after the SHA was read) fails with a non-refusal error, records nothing, and origin keeps the advanced tip.
  Update the file's header, which today says every case runs through `CheckRemoteBranchRequestForTest` and that no test there asserts a remote deletion, to describe these executor-level cases driven through `DeleteArchivedWeftBranchForTest`.

**ImpactSummary:** additive enum members, one struct field and one executor branch in `destroy.go`; existing remote-deletion call sites keep the unleased path.
