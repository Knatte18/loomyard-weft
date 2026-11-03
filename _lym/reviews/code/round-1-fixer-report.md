# Fixer-report, code round 1

`lym gate test ./internal/fabricengine --tier 0 --tier 1` passes after the last change.
F1 is a decision-class finding and went to the parent, so it is left out of this report.

## Changes

- **F3** fixed as proposed, in `internal/fabricengine/pushlockholder.go`.
  `describePushLockHolder` now reads a record with no positive pid or a zero `Since` as unreadable.
  The `EmptyObject` row of `TestDescribePushLockHolder` covers it.
- **F4** fixed as proposed, in `internal/fabricengine/pushlockholder.go`.
  The dead-pid clause now says no running process by that pid was found and the record may be stale, and drops the claim about the current holder.
  The `DeadHolder` row's want text and the doc comment match.
  The package doc in `internal/fabricengine/doc.go` now says "may be stale", found by the self-check.
- **F5** fixed as proposed, in the new `internal/fabricengine/pushlockholder_integration_test.go`.
  `TestTryRecordsLocks_HolderRecord` has a Weft row and a Bolt row, and `TestLockWeftForReplay_HolderRecord` covers the replay lock.
  Each weft case asserts the record names this process while the lock is held and is gone after the release.
  The Bolt row asserts no record in the worktree root or the weft lock directory.
- **F6** fixed as proposed, in `internal/fabricengine/pushlockholder_test.go`.
  `TestHoldPushLock_ReleasesTheLockWhenTheRecordFails` has two rows, a record path that is a non-empty directory and a lock directory that sits under a regular file.
  Both make the record write and its removal fail, and the test asserts the release returns nil and the lock can be taken again.
- **F2** left unchanged, see Deferred.

Commits: `38f1dae1` (F5), `bffc8b3c` (F3, F4 and F6 together, since all three edit `pushlockholder.go` and `pushlockholder_test.go`), and the self-check commit that follows them.

## Disputed

None.

## Deferred

- **F2**: the `PushTask` seam is `func() error` (`internal/landingshed/deps.go:358`) and carries no context, so no cancellable wait tied to the run can reach `pushRidingOutBusyLock` without changing the seam and its call sites in `landingshed`.
  The review's `fix` key names this case and says to leave the code and say so here.
  The retry still ends after its fixed pauses and returns the last busy error.

## Self-check

- fix-code-correctness, LOW: the Bolt row of `TestTryRecordsLocks_HolderRecord` looked only in the weft lock directory. Fixed: it now also checks the worktree root.
- fix-proposal, LOW: same Bolt-row gap. Merged with the one above and fixed.
- fix-proposal, LOW: `internal/fabricengine/doc.go` still said a dead-pid record "is stale". Fixed: it now says "may be stale".
- fix-consistency: no findings. It noted the doc.go wording above and that the test file header does not mention the direct holder test, and rated neither.
- fix-affected-set: no findings.
