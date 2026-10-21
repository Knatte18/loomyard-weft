# Verify gate failed on an unrelated reedcli test

The diff-scoped verify for a `planglyph`-only change ran `internal/reedcli`, and `TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping` failed on every rerun, although the package's test closure includes no changed file.
The watchdog consumes its stop marker only through an inotify watch, and the host sat at about 119 of 128 `fs.inotify.max_user_instances`, so a failed watch open leaves the marker unread and the stop subtests time out.
The writer can neither fix the host limit nor change an unrelated package in this task, so the finding went to the parent.
