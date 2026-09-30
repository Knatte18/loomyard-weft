# Card 5 — add-replace-archived-weft

**Edit:**
- `internal/fabricengine/add.go`
- `internal/fabricengine/export_test.go`
- `internal/fabricengine/add_remoteleftover_integration_test.go`
- `internal/fabricengine/remove_warpbranch_integration_test.go`
- `internal/fabricengine/doc.go`

**Prosa:**
- `CONSTRAINTS.md`

**Uses:**
- `plan:internal/fabricengine#weftLeftover`
- `plan:internal/fabricengine#probeWeftLeftover`
- `plan:internal/fabricengine#ownedPairWeftBranch`
- `plan:internal/fabricengine#dirtyArchivedOnRemote`
- `internal/fabricengine#deleteRemoteBranch`
- `internal/fabricengine#pushWeftBranch`
- `internal/fabricengine#Topology.rollbackAdd`

**Intent:**
Replace a provably-archived leftover of the pair's weft branch on origin, so `lyx fabric add <slug>` after a plain `lyx fabric remove <slug>` succeeds deterministically.
The pre-flight `weftLeftover` answer is carried to step 12; the replacement runs there, immediately before `pushWeftBranch`, so an Add failing earlier leaves the remote untouched.

In `add.go`, when the answer's tip is non-empty:
call the test hook (below) when set, then `deleteRemoteBranch` with a `remoteBranchRequest` on the weft repo root and `originRemoteName`, branch `weftBranch`, `ownedPairWeftBranch(l, warpBranch)`, `dirtyArchivedOnRemote(<the answer's tag>)`, `leaseSHA` set to the answer's tip, and `force: false`.
A refusal or a failed lease rolls back through `rollbackAdd` and returns an error naming the branch and wrapping the cause.
Rollback never restores the deleted remote branch: its content is reachable from the archive tag, and recreating it would re-block the next retry — say so in `rollbackAdd`'s doc comment.
The record then reads `remote_branch_deleted` followed by the weft `branch_pushed`, both existing Kinds.
The hook is an unexported package var, `addBeforeWeftReplaceHook func()`, nil in production;
`export_test.go` gains `SetAddBeforeWeftReplaceHookForTest(t testing.TB, fn func())`, which sets it and restores it in `t.Cleanup`, documented as usable only from a non-parallel test (the `spawnDetachedPushFn` precedent).

Tests in `add_remoteleftover_integration_test.go`:
- divergence: add, commit and push an extra weft commit, plain Remove, re-Add → succeeds; origin's weft branch equals the new local weft tip; the old tip is an ancestor of, or equal to, the target of an `archive/<slug>/*` tag on the weft bare; the second Add's record holds `remote_branch_deleted` for `<slug>-weft` before its weft `branch_pushed`.
- archived ancestor: add, commit an extra weft commit without pushing it, plain Remove (the archive tag lands on the unpushed commit, a strict descendant of origin's tip), re-Add → succeeds.
- warp fast-forwardable: add, plain Remove, commit on the prime so its HEAD advances past origin's workless warp branch, re-Add → succeeds and origin's warp branch equals the new warp tip.
- lease race (non-parallel): the hook pushes a commit onto origin's `<slug>-weft` from a clone → Add fails with an error naming the branch, no weft worktree remains, and origin keeps the moved tip.

In `remove_warpbranch_integration_test.go`, `TestRemove_AddSucceedsAfterWarpBranchDeleted` goes back to a plain Remove (`remote=false`), dropping the comment about the timestamp-dependent re-add gap.

Docs, same commit:
- `doc.go`: after the paragraph on `Remove`'s archiving, a paragraph on `Add`'s re-add handling — the pre-flight probes of both origins, the archive tag as the orphan proof, the leased gated replacement at step 12, the refusal outcomes, and that `SkipPush`/`SkipGit` skip all of it.
- `CONSTRAINTS.md`, the archive bullet of the Fabric Git Invariant: add a sentence that `Add` replaces a leftover remote weft branch only when an `archive/<slug>/*` tag covers its tip.

**ImpactSummary:** `Topology.Add` step 12 gains one gated, leased remote deletion; tests and docs within `internal/fabricengine`, plus one `CONSTRAINTS.md` sentence.
