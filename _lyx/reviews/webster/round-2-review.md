---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/hubreconcile/walk.go:snapshotConfig
    summary: snapshotConfig records any os.ReadFile error as an absent file, so a restore after a failed reconcile deletes a config file that exists but cannot be read
  - id: F2
    severity: LOW
    class: design
    location: internal/hubreconcile/walk.go:reconcileWorktree
    summary: a Commit error that follows a landed weft commit (PartialCommitError, correspondence not recorded) still restores the prior bytes, leaving the records worktree dirty against the new HEAD
  - id: F3
    severity: LOW
    class: scope
    location: internal/hubreconcile/ensure_integration_test.go
    summary: reconcileHubWide's restore after a failure has no test; only the per-worktree ReconcileAll failure and the pre-commit merge restore are reached
---

Range: `status.json` gives `product.parent` = `main`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
`go build ./...`, `go vet` over the three touched packages, and the untagged and `integration` tests of `hubreconcile` and `preflightshed` pass.

The four focus questions:

1. Restore on every failure after the write: settled for the `ReconcileAll` error, the merge-beginning-before-commit case and the `*WorktreeError` mapping; two gaps remain (F1, F2) and one is untested (F3).
2. `reconcileRefusal`: every branch is reachable from what `Ensure` produces and none is dead.
   `WorktreeError` with a file is reached by the unparseable-config integration test.
   `WorktreeError` with no file comes from a failed commit, and the generic branch from list-worktrees, lock-dir, lock-acquire, no-main-worktree and stamp-write errors; `TestReconcileRefusal_NamesTheCauseAndLoomResume` pins both.
   `LockTimeoutError` is reached by the held-lock integration test.
   Every message ends with `lyx loom resume` in the task worktree, and the file-bearing one also names the file and `lyx config <module>`.
   No defect.
3. `loom start` and `loom resume` through the built binary: `TestLoomResumeReconcilesTheHubConfigBeforeArming` runs the shared binary, asserts exit 1, the file path, `lyx config loom` and `re-run this command`, and that no stamp is written.
   No defect.
4. `snapshotConfig`: the dropped `data == nil` case was right to go, since `os.ReadFile` returns a non-nil slice for an empty file.
   The remaining `err != nil` case does occur, but it conflates two causes (F1).

### [MEDIUM] snapshotConfig treats an unreadable file as absent, and the restore deletes it

**Location:** `internal/hubreconcile/walk.go`, `snapshotConfig` and `restoreConfig`.
**Issue:** In the registry loop, every `os.ReadFile` error maps to `prior[path] = nil`, which `restoreConfig` reads as "the file was absent" and answers with `os.Remove`.
A module file that exists but cannot be read (permission denied, a directory at the path) is therefore recorded as absent.
`configsync.ReconcileAll` returns a `*FileError` on exactly that read error, so the very failure the restore exists for then deletes the operator's file.
The directory loop has the mirror gap: a listed file that fails to read is skipped silently, so a legacy file a migration later removes is never restored.
`ReadDir`'s error is discarded too.
Either way the claim in `doc.go` and in the `reconcileWorktree` comment, that a failed walk leaves the files at their prior bytes, does not hold.
**Fix:** Make `snapshotConfig` return an error, with `nil` only for `fs.ErrNotExist`.
Any other read or list error fails the worktree (or hub-wide) reconcile as a `*WorktreeError` naming the file, before anything is written.
Add a row to the existing failure test for an unreadable module file.

### [LOW] A restore after a landed commit leaves the records worktree dirty

**Location:** `internal/hubreconcile/walk.go`, `reconcileWorktree` (the `f.Commit` error branch) and `failRestoringConfig`.
**Issue:** The config files are weft files, so `Fabric.Commit` runs `commitWeftLocked`, which returns `committed == true` with an error when `RecordCorrespondence` fails after the commit landed (`*PartialCommitError`, `weftCommitted`).
`reconcileWorktree` discards the `CommitResult` and restores the prior bytes anyway.
The records worktree then differs from its own HEAD until a retry rewrites the files, and a retry in between sees a dirty tree.
This is the one post-write failure that does not leave the worktree as it was before the call.
**Fix:** Keep the `CommitResult` and skip the restore when `res.Committed()` is true, returning the `*WorktreeError` alone.

### [LOW] reconcileHubWide's restore is not reached by any test

**Location:** `internal/hubreconcile/ensure_integration_test.go`.
**Issue:** The first-review fix added a restore to `reconcileHubWide`, which runs on the board dir through `Bolt.CommitWritten`.
The tests reach the per-worktree restore (`TestEnsure_UnparseablePairConfigNamesPairAndFile`, `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles`) and the board push failure (`TestEnsure_UnreachableBoardRemoteStillCommitsHubWideConfig`), but none fails the hub-wide step after a hub-wide file was written.
`board` precedes `fabric` in the registry, so a committed unparseable `fabric.yaml` with a stale `board.yaml` reaches it.
**Fix:** Add one row or assertion to `TestEnsure_UnparseablePairConfigNamesPairAndFile`, or a sibling case in the same table, that breaks a hub-wide file, asserts the `*WorktreeError` names it, the board stays clean and no stamp is written.
