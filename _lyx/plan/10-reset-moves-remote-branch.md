# Card 10 — reset-moves-remote-branch

**Create:**
- `plan:internal/fabricengine#Fabric.RemoteOnlyCommits` -> `func (f *Fabric) RemoteOnlyCommits() (tip string, commits []string, err error)`

**Edit:**
- `internal/fabricengine/destroy.go`
- `internal/fabricengine/mutation.go`
- `internal/fabricengine/doc.go`
- `internal/webstercli/reset.go`
- `contracts/specs/refusal-spec.md`
- `internal/fabricengine/resetpairwarp_integration_test.go`
- `internal/fabricengine/livestate_mutationoracle_test.go`
- `internal/webstercli/reset_test.go`

**Uses:**
- `plan:internal/gitrepo#Repo.UpdateRemoteBranchLeased`
- `plan:internal/gitrepo#Repo.CommitsNotIn`

**Intent:**
Implement decision D7 (#404): `lyx webster reset` leaves the remote task branch consistent with the rewound local one, so a later push is not rejected.

`ResetPairWarp` in `destroy.go` gains a `SyncOptions` parameter and runs, in this order: the existing gate checks (containment, ownership, dirtiness; force stays false); then, unless skipped, a fetch of the remote and an ancestry check of the remote task branch against the local HEAD; then the lease-guarded remote update; then the checkout rewrite (`ResetHard`).
A refusal at any step before the remote update leaves both the remote and the local branch unchanged, and the first refusal an operator meets is the local, cheap one.
The remote update moves only the pair's own warp branch (`CurrentBranch`, re-checked against `parentBranch` exactly as `ownedPairWarpCheckout` does, so the parent branch is never updated), only to `sha`, through `UpdateRemoteBranchLeased` with the lease on the exact remote tip the ancestry check ran against.
It appends a new mutation kind, `KindRemoteBranchUpdated` (`remote_branch_updated`, declared in `mutation.go`, with the remote ref as target and `sha` as detail), to `rec` only once the push succeeded (`PATTERN-mutation-record`).
The update is skipped when the remote tip already equals `sha` or is an ancestor of it, when the remote has no such branch, when the repository has no remote configured, and under `SkipPush`; skipped means no fetch-dependent refusal and no record entry.

`RemoteOnlyCommits` fetches and returns the remote task branch's tip and the commits on it that the local HEAD lacks (`CommitsNotIn(tip, HEAD)`), with an empty tip when the remote has no such branch or no remote is configured; `ResetPairWarp`'s ancestry check and card 11's Publish halt both use it, so the two read the same answer.
All git runs through `gitrepo`/`gitexec`; `websterengine` stays fabric-blind.

Refusals, each returned before any mutation and each naming its way forward (`PATTERN-refusal-way-forward`; no denied command, `PATTERN-no-denied-recovery`):
remote tip holds commits the local HEAD lacks — the message lists those commits (SHA and subject) and names both cases: if they are the run's own abandoned commits, run `git merge --strategy ours origin/<task-branch>` in the task worktree, which takes no content and cannot conflict, then re-run `lyx webster reset --to <target>`, which rewinds both and discards what the merge brought in; if they may be someone else's work, do not re-run the reset, which would discard them, and leave the rewind to the operator; it offers no plain merge.
This covers the #404 state (a remote that diverged from a branch an earlier reset rewound) and a remote tip that moved after the fetch, which the lease turns into the same refusal.
A fetch or remote update that fails for another reason (network, authentication) — re-run the reset once the remote is reachable.
If the checkout rewrite fails after the remote moved, the error states that the remote task branch was already updated (the remote then sits at an ancestor of the local HEAD, so a later push fast-forwards) and names re-running the reset, which converges.

Bound, stated in `ResetPairWarp`'s doc comment: the remote update rewrites only the pair's own task branch, only to the commit the reset moves the local branch to, only when every commit it drops is reachable from the local HEAD, and with a lease on the remote tip the ancestry check saw; a remote-only commit becomes reachable only through the `ours` merge the operator runs after reading the listed commits, which is the one judgment left to the operator.

`internal/webstercli/reset.go` passes `fabricengine.EnvSyncOptions()`, prefixes the refusals as today, and its `Long` text says the reset also moves the remote task branch and lists the new refusals; the envelope's `mutations` carry the new entry.
Docs: `doc.go`'s `ResetPairWarp` paragraph; refusal-spec rows "reset: remote task branch holds commits HEAD lacks", "reset: remote unreachable" and "reset: checkout failed after the remote update", plus the `reset` mention in "What a way forward is" if it needs the remote half.

Tests (hubforge hub with its origin, `PATTERN-hubforge-fixtures`):
`resetpairwarp_integration_test.go`: after pushed commits, the reset leaves the remote branch at `sha` with a `remote_branch_updated` entry and a following push succeeds; a remote holding a commit the local HEAD lacks (including a local branch rewound by an earlier reset with the remote still holding the abandoned commits) refuses listing it, with remote and local unchanged, and after `git merge --strategy ours origin/<branch>` a re-run converges; a remote tip moved between fetch and update refuses with nothing changed; a dirty tracked path outside `ownPaths` refuses before any fetch, with the remote untouched; an unreachable remote URL refuses and changes nothing; a remote tip that is an ancestor of `sha`, a branch absent on the remote, a repository with no remote and `SkipPush` each leave the remote alone with no record entry.
Add the new kind to `livestate_mutationoracle_test.go`'s kind tables.
`internal/webstercli/reset_test.go`: drive `lyx webster reset --to start` through the cobra command after pushed batches and assert the remote task branch sits at the start commit and the envelope's mutations carry `remote_branch_updated`; with a remote-only commit it exits 1 naming the `ours` merge and changes nothing.
Existing `ResetPairWarp` callers in tests pass the new parameter.

**ImpactSummary:** ResetPairWarp gains a fetch, an ancestry gate and a leased remote update before its checkout rewrite, a new mutation kind and a shared remote-divergence read; webster reset's sole call site and its refusals follow.
