# lyx needs a newer git than 2.43: cherry-pick --empty=drop and Bolt rebase tests fail

`git-243-cherry-pick-empty` · Note · bug, fabric

Imported from [issue #743](https://github.com/Knatte18/loomyard/issues/743).

Seen on a Linux host with git 2.43.0 (Ubuntu 24.04's packaged git), go 1.26.9, tmux 3.4, running `lyx gate test --tags integration ./...` on main plus the #739 fix.

Failures:
- `internal/gitrepo` `TestCherryPick/a_pick_whose_change_is_already_carried_leaves_HEAD_unmoved`: `git cherry-pick --empty=drop <sha>` exits 129 with the usage text, since git 2.43 has no `--empty` for cherry-pick.
- `internal/fabricengine` `TestBolt_PullThenCommitWritten` and `TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict`: `git pull --rebase` stops on `could not apply ... lyx: seed stencils` instead of dropping the seed commit, and one case reports `skipped "diverged", want the genuine seed commit dropped and the write to run`.
- `internal/fabricengine` merge-lock test (`mergelock_integration_test.go:342` and `:365`): `merge preconditions failed: worktree dirty`, want `ErrForeignMergeState`; possibly the same git version cause, not confirmed.

The same calls run in production paths, so fabric and Bolt writes can fail on this host outside tests too.

Expected: lyx states its minimum git version and checks it at start with a clear error, or avoids flags the packaged git of a current Ubuntu LTS lacks.