# Publish verify failed on an unrelated tmux test

Publish failed on `internal/reedengine` `TestBoot_TmuxConf/AppendingFileIsSourcedOncePerServerStart/TwoWorktreesBootedConcurrently` with `run list-commands: exit status 1: server exited unexpectedly`.
The task's diff touches only `internal/creelcli` and `cmd/lyx/stencilseedgate_test.go`, so the failure reads as a race in the concurrent tmux server boot rather than a defect of this change.
The test carries the `tmux` build tag, and tmux-tier runs are refused in a darn session, so the writer could neither reproduce it nor confirm a fix and handed off with no new commit.
