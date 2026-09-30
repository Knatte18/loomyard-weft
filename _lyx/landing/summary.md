# fabric add: replace an archived leftover remote weft branch

`lyx fabric add <slug>` after a plain `lyx fabric remove <slug>` used to fail whenever the leftover `<slug>-weft` branch on the weft origin had diverged from the re-created one.
The push was rejected non-fast-forward in step 12, after the pair was built, and Add rolled back.
Whether it worked depended on commit timestamps, which is why `TestRemove_AddSucceedsAfterWarpBranchDeleted` flaked.

Add now decides before any mutation, and either succeeds deterministically or refuses with a working remedy.

## Behavior

- **Archived leftover:** when the remote weft tip equals or is an ancestor of the target of some `archive/<slug>/*` tag on origin, Add builds the new pair as before.
  It then deletes the leftover remote branch and pushes the new one.
  The old content stays reachable from the archive tag that Remove pushed.
- **Unarchived leftover:** Add refuses at pre-flight.
  The error names the branch and the remote tip, and offers three remedies: `lyx fabric remove <slug>` if a live pair holds the slug on another hub, `git push origin --delete <slug>-weft` once the content is disposable, or a different slug.
  The CLI reports it as a bare `output.Err` with no mutation record.
- **Adopt path** (local weft branch already exists): refuses at pre-flight unless the remote tip equals or is an ancestor of the local tip, so the step-12 push fast-forwards.
- **Warp side:** if origin already has the warp branch and its tip is not an ancestor of the warp HEAD the new branch forks from, Add refuses at pre-flight instead of failing at step 11.
  Fabric never deletes a remote warp branch, so the remedy is hand deletion or a different slug.
- `SkipPush` and `SkipGit` skip every probe and the replacement, as they skip both pushes.

## Design decisions

- **Archive tag as proof.**
  The ancestor test fetches the remote tip and the archive tag targets into `FETCH_HEAD`, creating no local refs, because a tag pushed from another hub has no local objects.
  A commit still missing after the fetch counts as unreachable.
- **Replace, never adopt.**
  The old branch carries stale `_lyx` run state that would mislead a fresh task.
- **Leased deletion.**
  The deletion is `--force-with-lease` against the SHA the pre-flight saw (new `gitrepo.DeleteRemoteBranchLeased`).
  A branch that moved in between fails the deletion, Add rolls back, and the moved tip stays on origin.
- **Timing.**
  The deletion runs immediately before the weft push, so an Add that fails earlier leaves the remote untouched.
- **Destruction chokepoint.**
  The deletion goes through `destroy.go`'s four-check pipeline.
  `remoteBranchRequest` gains an ownership kind (the pair's own weft branch), a dirtiness kind (archived on the remote) and a lease SHA.
  The mutation record gains `remote_branch_deleted` then `branch_pushed`, both existing Kinds.
- **Rollback** after a completed replacement does not restore the old remote branch: its content is in the archive tag, and recreating it would re-block the retry.
- `Remove` is unchanged: it still leaves remote branches alone unless `--remote` is given.

Docs: `internal/fabricengine/doc.go` (Add's section) and the archive bullet of `CONSTRAINTS.md`'s Fabric Git Invariant.
`TestRemove_AddSucceedsAfterWarpBranchDeleted` is back to a plain Remove, with new integration tests for each outcome above and unit tests for the extended `destroy.go` request.

## To check by hand

- The plan-level integration suite failed, and the triage did not attribute it to this change.
  It lists one pre-existing failure, already failing at the starting commit: `internal/fabriccli.TestRunCLI_CleanupRemoteFailureExitsNonZero`.
  No test is flagged flaky.
  Confirm that failure is unrelated to the `internal/fabriccli` change here.
- Run `go test ./...` and the integration-tagged suite once on a clean checkout.
- Against a real branch-protected origin, confirm the replacement's rejected deletion rolls Add back with the remote's message.
- A second hub holding a live pair of the same slug at an archived tip loses its remote weft branch on replacement.
  Its content stays in the archive tag, and its next push is rejected rather than diverging.
