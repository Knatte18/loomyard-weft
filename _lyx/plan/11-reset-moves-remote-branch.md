# Card 11 — reset-moves-remote-branch

**Edit:**
- `internal/fabricengine/destroy.go`
- `internal/fabricengine/mutation.go`
- `internal/fabricengine/doc.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/reset.go`
- `contracts/specs/refusal-spec.md`
- `internal/fabricengine/resetpairwarp_integration_test.go`
- `internal/fabricengine/livestate_mutationoracle_test.go`
- `internal/fabricengine/export_test.go`
- `internal/webstercli/reset_test.go`
- `cmd/lyx/destructiveguard_test.go`

**Uses:**
- `plan:internal/gitrepo#Repo.UpdateRemoteBranchLeased`
- `plan:internal/gitrepo#ErrLeaseRejected`
- `plan:internal/fabricengine#Fabric.RemoteOnlyCommits`

**Intent:**
Implement decision D7 (#404): `lyx webster reset` leaves the remote task branch consistent with the rewound local one, so a later push is not rejected.

`ResetPairWarp` in `destroy.go` gains a `SyncOptions` parameter and runs, in this order: the existing gate checks (containment, ownership, dirtiness; force stays false); then, unless skipped, a fetch of the remote and an ancestry check of the remote task branch against the local HEAD; then the lease-guarded remote update; then the checkout rewrite (`ResetHard`).
A refusal at any step before the remote update leaves both the remote and the local branch unchanged, and the first refusal an operator meets is the local, cheap one.
The remote update is a new executor in `destroy.go`, `updateRemoteBranch(rec *Mutations, req remoteBranchUpdateRequest)`, beside `deleteRemoteBranch`: it takes the leading `rec` and runs the same gate pipeline (`checkPathRequest` over the pair-warp request, in the containment, ownership, dirtiness, force order) before it pushes, so the destructive-chokepoint invariant holds for the new force push.
`ResetPairWarp` therefore runs the gate once up front (a refusal here is the first, local refusal), then the fetch and the ancestry read, then the executor, then `resetHardTo`, which re-runs the same gate before the checkout rewrite.
It moves only the pair's own warp branch (`CurrentBranch`, re-checked against `parentBranch` exactly as `ownedPairWarpCheckout` does, so the parent branch is never updated), only to `sha`, through `UpdateRemoteBranchLeased` with the lease on the exact remote tip the ancestry check ran against.
It appends a new mutation kind, `KindRemoteBranchUpdated` (`remote_branch_updated`, declared in `mutation.go`, with the remote ref as target and `sha` as detail), to `rec` only once the push succeeded (`PATTERN-mutation-record`).
`cmd/lyx/destructiveguard_test.go` is the guard that keeps a destructive primitive inside `destroy.go`: it gains `.UpdateRemoteBranchLeased(` as a banned token (beside `.DeleteRemoteBranch(`) and a `destructiveGuardRecordingExecutors` row for `updateRemoteBranch`, and its floor comment is reworded so it counts no rows.
`destroy.go`'s file comment, which pins "every one of the nine executors", is reworded the same way.
The same tallies are pinned elsewhere and go false with the new executor, token and kind, so each is reworded to count nothing: `cmd/lyx/destructiveguard_test.go`'s header ("nine banned tokens", "destroy.go's nine executors") and the comment above `TestNoDestructiveBypass_FabricengineProductionSource` ("the nine construction/call tokens"), and `internal/fabricengine/doc.go`'s "nine gate executors auto-record eight of the nineteen kinds".
The update is skipped when the remote tip already equals `sha` or is an ancestor of it, when the remote has no such branch, when the repository has no remote configured, and under `SkipPush`; skipped means no fetch-dependent refusal and no record entry.
`SkipGit` does not skip the fetch or the remote update, because D7 names `SkipPush` as the only bypass; `internal/webstercli/reset_test.go`'s fixture sets `WEFT_SKIP_GIT=1` on every test, so the new CLI rows run the remote update under it.
This makes the remote update the one push-shaped fabric step that ignores `SkipGit`, unlike the other pushes, which bypass on `SkipGit || SkipPush`.

`ResetPairWarp`'s ancestry check reads the remote task branch's tip and the commits the local HEAD lacks through `Fabric.RemoteOnlyCommits`, so it and the Publish halt read the same answer.
All git runs through `gitrepo`/`gitexec`; `websterengine` stays fabric-blind.

Refusals, each returned before any mutation and each naming its way forward (`PATTERN-refusal-way-forward`; no denied command, `PATTERN-no-denied-recovery`):
remote tip holds commits the local HEAD lacks — the message lists those commits (SHA and subject) and names both cases: if they are the run's own abandoned commits, run `git merge --strategy ours origin/<task-branch>` in the task worktree, which takes no content and cannot conflict, then re-run `lyx webster reset --to <target>`, which rewinds both and discards what the merge brought in; if they may be someone else's work, do not re-run the reset, which would discard them, and leave the rewind to the operator; it offers no plain merge.
This covers the #404 state (a remote that diverged from a branch an earlier reset rewound) and a remote tip that moved after the fetch: `updateRemoteBranch` maps `gitrepo.ErrLeaseRejected` to this same refusal, with the way forward unchanged.
A fetch that fails, or a remote update that fails with any error other than `ErrLeaseRejected` (network, authentication, a hook rejection) — re-run the reset once the remote is reachable.
If the checkout rewrite fails after the remote moved, the error states that the remote task branch was already updated (the remote then sits at an ancestor of the local HEAD, so a later push fast-forwards) and names re-running the reset, which converges.

Bound, stated in `ResetPairWarp`'s doc comment: the remote update rewrites only the pair's own task branch, only to the commit the reset moves the local branch to, only when every commit it drops is reachable from the local HEAD, and with a lease on the remote tip the ancestry check saw; a remote-only commit becomes reachable only through the `ours` merge the operator runs after reading the listed commits, which is the one judgment left to the operator.

`internal/webstercli/reset.go` passes `fabricengine.EnvSyncOptions()`, prefixes the refusals as today, and its `Long` text says the reset also moves the remote task branch and lists the new refusals; the envelope's `mutations` carry the new entry.
When `ResetPairWarp` fails after `rec` already holds `remote_branch_updated` (the checkout-failed case), the verb is no longer a pre-flight failure (`PATTERN-mutation-record`): its error envelope carries `mutations` from `rec` and `partial: true`, and the message does not open with "refused", since the remote was changed.
Every refusal before the remote update keeps the bare error envelope.
Docs: `fabricengine/doc.go`'s `ResetPairWarp` paragraph, which also states that `SkipGit` does not gate the remote update, and its closed `Kind` vocabulary list, which gains `remote_branch_updated`; `internal/websterengine/doc.go`'s `lyx webster reset` paragraph, whose envelope sentence ("the `worktree_reset` entry", `partial` always false) and bound sentence now cover the remote update and the partial envelope; refusal-spec rows "reset: remote task branch holds commits HEAD lacks", "reset: remote unreachable" and "reset: checkout failed after the remote update", plus the `reset` mention in "What a way forward is" if it needs the remote half.

Tests (hubforge hub with its origin, `PATTERN-hubforge-fixtures`):
`resetpairwarp_integration_test.go`: extend `TestResetPairWarp_DiscardsCommitsAndOwnPathDirtKeepsUntracked` (pushing the commits first on a hub with its origin) so the reset leaves the remote branch at `sha` with a `remote_branch_updated` entry and a following push succeeds; extend `TestResetPairWarp_DirtyPathOutsideOwnPathsRefuses` to assert the refusal leaves the remote untouched and is met before any fetch; add one new table-driven test for the rest: a remote holding a commit the local HEAD lacks (including a local branch rewound by an earlier reset with the remote still holding the abandoned commits) refuses listing it, with remote and local unchanged, and after `git merge --strategy ours origin/<branch>` a re-run converges; a remote tip moved between fetch and update refuses with the divergence way forward and nothing changed (the `ErrLeaseRejected` path, reached through a driver in `export_test.go` beside `DeleteArchivedWeftBranchForTest` that runs `updateRemoteBranch` with a lease SHA stale against the origin, since `ResetPairWarp` leaves no window between its fetch and its update, and asserting the refusal, its way forward, an unchanged remote and no record entry); an update that fails for a non-lease reason (a `pre-receive` hook on the origin that exits non-zero) refuses with the re-run-once-reachable way forward and nothing changed; an unreachable remote URL refuses the same way and changes nothing; a remote tip that is an ancestor of `sha`, a branch absent on the remote, a repository with no remote and `SkipPush` each leave the remote alone with no record entry;
a checkout rewrite forced to fail after the remote update (the implementer picks the injection, for example a held `index.lock` in the warp's git directory) returns an error stating the remote task branch was already updated and naming the re-run, leaves `remote_branch_updated` in `rec`, and a re-run converges.
Add the new kind to `livestate_mutationoracle_test.go`'s kind tables, and the executor and banned-token entries to `cmd/lyx/destructiveguard_test.go`.
`internal/webstercli/reset_test.go`: drive `lyx webster reset --to start` through the cobra command after pushed batches and assert the remote task branch sits at the start commit and the envelope's mutations carry `remote_branch_updated`; with a remote-only commit it exits 1 naming the `ours` merge and changes nothing; with the checkout forced to fail after the remote update it exits 1 with an envelope carrying `remote_branch_updated` and `partial: true`.
Existing `ResetPairWarp` callers in tests pass the new parameter.

**ImpactSummary:** ResetPairWarp gains a fetch, an ancestry gate and a leased remote update, as a new gated executor, before its checkout rewrite, a new mutation kind and a partial envelope when the checkout fails after the remote moved; webster reset's sole call site, its refusals and the destructive guard's tables follow.
