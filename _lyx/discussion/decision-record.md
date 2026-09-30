# Decision record — fabric add after remove: leftover remote weft branch

## Goal

`lyx fabric add <slug>` after a plain `lyx fabric remove <slug>` (no `--remote`) succeeds deterministically when the leftover `<slug>-weft` branch on the weft origin is covered by an `archive/<slug>/*` tag and any leftover remote warp branch is fast-forwardable.
Every other leftover produces a clean pre-flight refusal with a working remedy, never a mid-Add push rejection followed by rollback.
The integration test that exposed the gap pins these outcomes deterministically, independent of commit timestamps.

## Scope

In:

- `Topology.Add` (`internal/fabricengine/add.go`): a pre-flight probe of the weft origin for the pair's weft branch, and a gated replacement of a provably-archived leftover before the weft push.
- A pre-flight fast-forward probe on the adopt path (the local weft branch already exists) against its remote copy.
- A pre-flight probe of the warp origin for the pair's warp branch, refusing when the new warp branch could not fast-forward it.
- One extension of the gated remote-branch deletion in `internal/fabricengine/destroy.go`: a new ownership kind (the pair's own weft branch), a new dirtiness kind (archived on the remote), and a lease SHA.
- Tests in `internal/fabricengine`, including rewriting `TestRemove_AddSucceedsAfterWarpBranchDeleted` back to a plain Remove.
- Docs: `internal/fabricengine/doc.go` (Add's section) and the archive bullet of `CONSTRAINTS.md`'s Fabric Git Invariant.

Out:

- Changing `Remove`: its default stays "leave remote branches alone"; `--remote` stays opt-in.
- Adopting the old remote weft branch's content into the new pair.
- Deleting or force-pushing any remote warp branch.
- The rollback's existing warp-branch residue: after a failure at or past step 11, the warp branch already pushed stays on origin, and under the default empty prefix the local bare-slug warp branch stays too (`ErrBranchExists` names its remedy).
- The webster integration-verify report carrying only `status: FAILED` without the failing test's name (a `websterengine` reporting gap, separate task).

## Decisions

- **The archive tag is the orphan proof.**
  Remove already pushes `archive/<slug>/<tip>` to the weft origin before tearing down (Fabric Git Invariant).
  A remote `<slug>-weft` whose tip equals, or is an ancestor of, the target of some `archive/<slug>/*` tag on origin is the leftover of a removed pair whose content is preserved.
  The probe compares SHAs first; for the ancestor test it fetches the remote branch tip and the listed archive tag targets into `FETCH_HEAD` without creating local refs, since a tag pushed from another hub has no local objects.
  A commit still missing locally after the fetch counts as unreachable.
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
  - local weft branch exists (today's adopt path) → refuse unless the remote tip equals or is an ancestor of the local weft tip (`merge-base --is-ancestor`, a missing object counting as not an ancestor), so the step-12 push fast-forwards;
  - remote tip archived → replace at step 12;
  - remote tip not reachable from any `archive/<slug>/*` tag → refuse.
  Rationale: a refusal before the first mutation has an empty record and nothing to roll back.
- **The weft refusal names the branch, the remote tip, and remedies that work whether or not a pair still holds the slug.**
  Remedies: if a pair of that slug is live on some hub, remove it there (`lyx fabric remove <slug>`, which archives it); if no hub holds the pair, delete the remote branch by hand once its content is known to be disposable (`git push origin --delete <slug>-weft`); or pick a different slug.
  The adopt-path refusal names the branch and the same hand-deletion and different-slug remedies.
  Every refusal is a typed error the CLI surfaces as a pre-flight failure (bare `output.Err`, no `mutations`/`partial` keys).
  Rationale: an unarchived remote branch may be a live pair on another hub, or a leftover from a Remove that ran before archive tagging existed; fabric must not decide that unattended.
- **Warp side: refuse at pre-flight, never delete.**
  If origin already has the warp branch and its tip is not an ancestor of (or equal to) the warp HEAD the new branch forks from, Add refuses before creating anything, naming the branch and the remedies (delete the remote branch by hand once its work is landed, or pick a different slug).
  The check is `ls-remote` plus a local `merge-base --is-ancestor`; an object missing locally counts as "not an ancestor".
  Rationale: today that case fails at step 11 after the pair is built; fabric deliberately performs no remote warp deletion ("fabric: no remote/GitHub branch deletion"), so refusing is the only in-policy outcome.
- **No probe when the push is suppressed.**
  `SkipPush` or `SkipGit` skips every pre-flight probe and the replacement, exactly as it skips both pushes.
  Rationale: those options promise Add touches no remote.
- **The deletion extends `remoteBranchRequest` in `destroy.go` and records `remote_branch_deleted`.**
  Ownership: a new closed-enum ownership kind, "the pair's own weft branch": the branch is exactly `WeftBranchName(warpBranch)`, weft-suffixed, and not the repo's primary weft branch.
  It has no local-checkout test, because at step 12 the same-named local branch is the replacement, checked out at the new weft worktree; the existing managed-branch kind refuses exactly that, and accepts any `branch_prefix` branch besides.
  Dirtiness: a new closed-enum kind, "archived on the remote", answered by the pre-flight's reachability proof and the lease SHA.
  The mutation record gains `remote_branch_deleted` then `branch_pushed`, both existing Kinds, in execution order.
  Rationale: Fabric Destruction Chokepoint Invariant — every destructive primitive runs the four-check pipeline in `destroy.go`; reusing existing Kinds keeps the closed `Kind` enum unchanged.
- **Rollback after a completed replacement does not restore the old remote branch.**
  Rationale: its content is reachable from the archive tag; recreating a ref Add deliberately deleted would re-block the next retry.

## Constraints

- `CONSTRAINTS.md` Fabric Destruction Chokepoint Invariant: the remote deletion extends `remoteBranchRequest` with the new ownership kind, dirtiness kind and lease SHA, and runs through `destroy.go`'s pipeline; a refusal is never discarded.
- Fabric Git Invariant: all git in `fabricengine`; the archive rule gains a sentence stating Add replaces a leftover remote weft branch only when an archive tag covers its tip.
- Mutation Record Invariant: pre-flight refusals carry no record; the replacement records only what observably changed.
- gitrepo Client Boundary / gitexec Checked-Call Invariant: remote probes, fetches and the leased deletion go through `gitexec`.
- Hermetic Git Test Environment and hubforge Fabric-Fixture Invariants: tests build hubs via `newFabricFixture`.
- Test Tier Purity: the new tests are integration-tagged, alongside `remove_warpbranch_integration_test.go`.

## Auto-mode assumptions

- A re-added slug is a fresh task, never a resume of the removed one.
- One slug is live on at most one hub at a time; the archive proof plus the lease is sufficient protection for the rare violation.
- An `archive/<slug>/*` tag on origin is only ever created by fabric's own teardown.

## Open risks

- A second hub holding a live pair of the same slug at a tip equal to or an ancestor of an archive tag's target loses its remote weft branch on replacement; its content stays reachable from the archive tag, and its next push is rejected rather than silently diverging.
- A weft origin that rejects branch deletion (branch protection) makes the replacement fail after the pair is built; Add rolls back and the error names the remote's refusal.
- The ancestor tests fetch the remote branch tip and the archive tag targets into `FETCH_HEAD`, as `resolveArchiveTip` already does for a branch; the plan must keep those fetches from creating local refs.

## Acceptance criteria

- `TestRemove_AddSucceedsAfterWarpBranchDeleted` uses a plain Remove (`remote=false`) again, and its re-add succeeds on every run.
- Deterministic divergence: add a pair, commit and push an extra weft commit so the old remote tip cannot equal the new one, plain-Remove, re-Add → succeeds; origin's weft branch equals the new local weft tip; the old tip is still reachable from its `archive/<slug>/*` tag on origin; the Add mutation record contains `remote_branch_deleted` followed by `branch_pushed` for the weft branch.
- Archived ancestor: the old remote tip is a strict ancestor of the archive tag's target (an unpushed local weft commit archived at Remove) → re-Add succeeds.
- Unarchived leftover: after a plain Remove, push a commit onto origin's weft branch directly (not reachable from any archive tag) → Add refuses before any mutation; the error names the branch, `lyx fabric remove <slug>` for a live pair, and the hand-deletion remedy; no warp or weft worktree, branch, portal or launcher is left; origin's weft branch is unchanged.
- Adopt path: the local weft branch exists and origin's weft branch holds a commit it lacks → Add refuses before any mutation, naming the branch.
- Lease race: a test-only hook that `Add` calls after pre-flight and before the deletion pushes a commit onto origin's weft branch → the deletion fails, Add rolls back the weft side, origin keeps the moved tip.
- Warp leftover: add a pair, commit and push warp work, squash-land it on the parent, plain-Remove (the local warp branch is deleted), re-Add → Add refuses before any mutation, naming the warp branch; no local warp branch is created.
- Warp fast-forwardable leftover: origin's warp branch is an ancestor of the parent HEAD → Add succeeds and the push fast-forwards.
- `SkipPush`: a re-add with an unarchived remote leftover succeeds locally and touches no remote.
- `destroy.go` unit tests cover the extended request shape: unset ownership or dirtiness refused; a branch other than the pair's own weft branch refused on ownership; the pair's weft branch checked out locally not refused on ownership; an unarchived tip refused on dirtiness; a stale lease SHA makes the deletion fail and leaves the remote ref unchanged.
- `go test ./...` and the integration-tagged suite pass; help-tree and constraint tests stay green.

## Notes for the plan writer

- `archive.go`'s `resolveArchiveTip` already does `ls-remote` then `fetch origin refs/heads/<branch>` into `FETCH_HEAD`; the archive reachability probe can follow the same shape, listing `refs/tags/archive/<slug>/*` with `ls-remote --tags`.
- `destroy.go` already has `remoteBranchRequest`, `checkRemoteBranchRequest` and `deleteRemoteBranch`; the new ownership and dirtiness kinds and a lease SHA extend that shape, and `gitrepo`'s `DeleteRemoteBranch` needs a leased variant.
- Add's pre-flight block (before `createGitWorktree`) already holds the "branch exists" and "no remote configured" refusals; the probes belong there.
