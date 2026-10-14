## Changes

- F1: fixed as proposed.
  `internal/loomcli/smoke_test.go` gains `waitRecordsIndexLockGone`, which resolves the path `git rev-parse --git-path index.lock` names in the records worktree and polls until the file is absent, failing after a deadline.
  `killDriversAndClearLog` calls it with a 5s deadline after `waitRunLockFree`.
  `lyx loom fmt` and `go vet` are clean; the tmux-tier smoke test was not run, since such runs are refused in this session.

## Disputed

None.

## Deferred

None.
