# Decision record — fabric add after remove: leftover remote weft branch

## Goal

`lyx fabric add <slug>` after a plain `lyx fabric remove <slug>` (no `--remote`) succeeds deterministically instead of failing on the leftover `<slug>-weft` branch on the weft origin.
A leftover remote branch that fabric cannot prove is safe to replace produces a clean pre-flight refusal with a remedy, never a mid-Add push rejection followed by rollback.
The integration test that exposed the gap pins both outcomes deterministically, independent of commit timestamps.

## Scope

In:

- `Topology.Add` (`internal/fabricengine/add.go`): a pre-flight probe of the weft origin for the pair's weft branch, and a gated replacement of a provably-archived leftover before the weft push.
- A pre-flight probe of the warp origin for the pair's warp branch, refusing when the new warp branch could not fast-forward it.
- One new gated remote-branch deletion shape in `internal/fabricengine/destroy.go`: ownership by the weft suffix, dirtiness answered by archive reachability, deletion bound to the observed remote SHA (lease).
- Tests in `internal/fabricengine`, including rewriting `TestRemove_AddSucceedsAfterWarpBranchDeleted` back to a plain Remove.
- Docs: `internal/fabricengine/doc.go` (Add's section) and the archive bullet of `CONSTRAINTS.md`'s Fabric Git Invariant.

Out:

- Changing `Remove`: its default stays "leave remote branches alone"; `--remote` stays opt-in.
- Adopting the old remote weft branch's content into the new pair.
- Deleting or force-pushing any remote warp branch.
- The webster integration-verify report carrying only `status: FAILED` without the failing test's name (a `websterengine` reporting gap, separate task).

## Decisions

- **The archive tag is the orphan proof.**
  Remove already pushes `archive/<slug>/<tip>` to the weft origin before tearing down (Fabric Git Invariant).
  A remote `<slug>-weft` whose tip is reachable from (equal to, or an ancestor of) the target of some `archive/<slug>/*` tag on origin is the leftover of a removed pair whose content is preserved.
  Rationale: deterministic, needs no new record or state from Remove, and proves no work is lost by replacing the branch.
- **Add replaces a provably-archived leftover; it never adopts it.**
  Add creates the new weft branch from the parent's weft branch as it does today, then deletes the leftover remote branch through the gated executor and pushes the new branch with the existing `pushWeftBranch`.
  Rationale: a re-add starts a fresh task; the old branch carries stale `_lyx` run state (shed status, discussion, origin record with the old parent) that would mislead the new run.
- **The deletion is leased to the SHA the pre-flight observed.**
  If the remote branch moved between probe and deletion, the deletion fails, Add rolls back, and the error names the branch.
  Rationale: closes the race where a live pair elsewhere pushes after the probe; that push would otherwise be lost unarchived.
- **The deletion runs immediately before the weft push (step 12), not at pre-flight.**
  Rationale: an Add that fails earlier leaves the remote untouched; the probe result is carried forward to the deletion.
- **Pre-flight weft outcomes, decided before any mutation:**
  - remote branch absent → today's path;
  - local weft branch exists (today's adopt path) → no probe change; the adopted branch already tracks or matches whatever is remote;
  - remote tip archived → replace at step 12;
  - remote tip not reachable from any `archive/<slug>/*` tag → refuse.
  Rationale: a refusal before the first mutation has an empty record and nothing to roll back.
- **The weft refusal names the branch, the remote tip, and the remedies.**
  Remedies: remove the pair on the hub that owns it (`lyx fabric remove <slug>`, which archives it), or pick a different slug.
  The error is a typed refusal the CLI surfaces as a pre-flight failure (bare `output.Err`, no `mutations`/`partial` keys).
  Rationale: an unarchived remote branch may be a live pair on another hub; fabric must not decide that unattended.
- **Warp side: refuse at pre-flight, never delete.**
  If origin already has the warp branch and its tip is not an ancestor of (or equal to) the warp HEAD the new branch forks from, Add refuses before creating anything, naming the branch and the remedies (delete the remote branch by hand once its work is landed, or pick a different slug).
  The check is `ls-remote` plus a local `merge-base --is-ancestor`; an object missing locally counts as "not an ancestor".
  Rationale: today that case fails at step 11 after the whole pair is built, and the rollback cannot delete a bare-slug warp branch under the default empty prefix, so the leftover local branch then blocks every retry with `ErrBranchExists`.
  Fabric deliberately performs no remote warp deletion ("fabric: no remote/GitHub branch deletion"), so refusing is the only in-policy outcome.
- **No probe when the push is suppressed.**
  `SkipPush` or `SkipGit` skips both pre-flight probes and the replacement, exactly as it skips both pushes.
  Rationale: those options promise Add touches no remote.
- **The new gated shape lives in `destroy.go` and records `remote_branch_deleted`.**
  Ownership: the weft-suffix managed-branch kind the existing remote deletion uses.
  Dirtiness: a new closed-enum kind, "archived on the remote", answered by the pre-flight's reachability proof and the lease SHA.
  The mutation record gains `remote_branch_deleted` then `branch_pushed`, both existing Kinds, in execution order.
  Rationale: Fabric Destruction Chokepoint Invariant — every destructive primitive runs the four-check pipeline in `destroy.go`; reusing existing Kinds keeps the closed `Kind` enum unchanged.
- **Rollback after a completed replacement does not restore the old remote branch.**
  Rationale: its content is reachable from the archive tag; recreating a ref Add deliberately deleted would re-block the next retry.

## Constraints

- `CONSTRAINTS.md` Fabric Destruction Chokepoint Invariant: the remote deletion is a new request shape through `destroy.go`'s pipeline; a refusal is never discarded.
- Fabric Git Invariant: all git in `fabricengine`; the archive rule gains a sentence stating Add replaces a leftover remote weft branch only when an archive tag covers its tip.
- Mutation Record Invariant: pre-flight refusals carry no record; the replacement records only what observably changed.
- gitrepo Client Boundary / gitexec Checked-Call Invariant: remote probes, fetches and the leased deletion go through `gitexec`.
- Hermetic Git Test Environment and hubforge Fabric-Fixture Invariants: tests build hubs via `newFabricFixture`.
- Test Tier Purity: the new tests are integration-tagged, alongside `remove_warpbranch_integration_test.go`.

## Auto-mode assumptions

- A re-added slug is a fresh task, never a resume of the removed one.
- One slug is live on at most one hub at a time; the archive proof plus the lease is sufficient protection for the rare violation.
- An `archive/<slug>/*` tag on origin is only ever created by fabric's own teardown.
- On the existing adopt path (the local weft branch already exists), that branch already matches or fast-forwards its remote copy, so the adopt path needs no probe.
- The warp pre-flight is in scope because it is the same re-add-after-remove failure on the same verb, and it is a probe plus refusal with no new destructive primitive.

## Open risks

- A second hub holding a live pair of the same slug at an archived tip loses its remote weft branch on replacement; its content stays reachable from the archive tag, and its next push is rejected rather than silently diverging.
- A weft origin that rejects branch deletion (branch protection) makes the replacement fail after the pair is built; Add rolls back and the error names the remote's refusal.
- `git merge-base --is-ancestor` with a remote tip object absent locally needs a fetch of the remote branch into `FETCH_HEAD` first, as `resolveArchiveTip` already does; the plan must keep that fetch from creating local refs.

## Acceptance criteria

- `TestRemove_AddSucceedsAfterWarpBranchDeleted` uses a plain Remove (`remote=false`) again, and its re-add succeeds on every run.
- Deterministic divergence: add a pair, commit and push an extra weft commit so the old remote tip cannot equal the new one, plain-Remove, re-Add → succeeds; origin's weft branch equals the new local weft tip; the old tip is still reachable from its `archive/<slug>/*` tag on origin; the Add mutation record contains `remote_branch_deleted` followed by `branch_pushed` for the weft branch.
- Unarchived leftover: after a plain Remove, push a commit onto origin's weft branch directly (not reachable from any archive tag) → Add refuses before any mutation; the error names the branch and the remedies; no warp or weft worktree, branch, portal or launcher is left; origin's weft branch is unchanged.
- Lease race: the remote weft branch moves between the pre-flight probe and the deletion → the deletion fails, Add rolls back, origin keeps the moved tip.
- Warp leftover: add a pair, commit and push warp work, squash-land it on the parent, plain-Remove (the local warp branch is deleted), re-Add → Add refuses before any mutation, naming the warp branch; no local warp branch is created.
- Warp fast-forwardable leftover: origin's warp branch is an ancestor of the parent HEAD → Add succeeds and the push fast-forwards.
- `SkipPush`: a re-add with an unarchived remote leftover succeeds locally and touches no remote.
- `destroy.go` unit tests cover the new request shape: unset ownership or dirtiness refused; a branch without the weft suffix refused on ownership; an unarchived tip refused on dirtiness.
- `go test ./...` and the integration-tagged suite pass; help-tree and constraint tests stay green.

## Notes for the plan writer

- `archive.go`'s `resolveArchiveTip` already does `ls-remote` then `fetch origin refs/heads/<branch>` into `FETCH_HEAD`; the archive reachability probe can follow the same shape, listing `refs/tags/archive/<slug>/*` with `ls-remote --tags`.
- `destroy.go` already has `remoteBranchRequest`, `checkRemoteBranchRequest` and `deleteRemoteBranch`; the new dirtiness kind and a lease SHA extend that shape, and `gitrepo`'s `DeleteRemoteBranch` needs a leased variant.
- Add's pre-flight block (before `createGitWorktree`) already holds the "branch exists" and "no remote configured" refusals; the two probes belong there.
