# Diff-scoped verify gate runs an unrelated package whose test depends on host inotify limits

`verify-diff-scope-unrelated-pkg` · Note · bug, gate

Imported from [issue #785](https://github.com/Knatte18/loomyard/issues/785).

The verify gate for a change confined to `internal/planglyph` ran `internal/reedcli` tests.
`TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping` failed on every rerun, although the package's test closure contains no changed file.

The test consumes the watchdog's stop marker only through an inotify watch.
The host was near its `fs.inotify.max_user_instances` limit (about 119 of 128), so a failed watch open leaves the marker unread and the stop subtests time out.

Two things are abnormal:

- The diff-scoped verify selected a package with no changed file in its closure, so an unrelated package can block a task.
- The test fails on host inotify exhaustion instead of skipping or reporting the cause, so the failure reads as a code defect.

The writer could neither raise the host limit nor change an unrelated package in the task, so the finding went to the parent.

What to do: check why the diff-scoping selected `internal/reedcli` for a `planglyph`-only diff, and make the watchdog test skip or fail with a clear message when the inotify watch cannot be opened.

Provenance: task `create-target-transient`, note `Darn-Write.md`.
