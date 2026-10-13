# Fixer report — discussion, round 2

## Changes

- F1 (MEDIUM), fixed as proposed, first option.
  Premise checked: `internal/lock/lock.go` wraps gofrs/flock, which holds the lock file open for the operation's length.
  D7 now retries a late-writer error: `syscall.ENOTEMPTY` on Linux and macOS, and on Windows `windows.ERROR_DIR_NOT_EMPTY` or `windows.ERROR_SHARING_VIOLATION` for a file under the target still open by its writer.
  The retry sentence and the Bound say "late-writer" in place of "not-empty".
  A new acceptance criterion covers the Windows sharing-violation retry.
- F2 (LOW), fixed as proposed: the recognition sentence's first half is the branch condition (git exited nonzero, the target is no longer a registered linked worktree and is still on disk), and the sentence states that neither half reads git's stderr or any message text.
- F3 (LOW), fixed as proposed.
  Premise checked in `internal/fabricengine/destroy.go`: `removeContainedPath` appends nothing, `removePath` appends `KindPathRemoved`, and `removeGitWorktree` appends `KindWorktreeRemoved` only on success.
  D7 now has a successful leftover removal append `KindPathRemoved` for the target with detail `recursive`, and Constraints names `PATTERN-mutation-record` for D7.
  The D7 acceptance criterion also checks that the record holds the entry.

The support log's Review rounds section records the round.

## Disputed

None.

## Deferred

None.
