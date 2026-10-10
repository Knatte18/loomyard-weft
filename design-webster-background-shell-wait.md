# webster: wait stops on a background shell at turn end and leaves it running

`webster-background-shell-wait` · Note · bug

Imported from [issue #496](https://github.com/Knatte18/loomyard/issues/496).

A `webster-background-shell` note reported that Master's wait stopped waiting on a still-running background shell at a turn end.
The wait counts the turn end after `background_shell_wait_min` (10 minutes) for a shell only the transcript reports, and at once for one the Stop payload reports when Master's output files exist.
lyx did not stop the shell; it ended only because shuttle removes Master's strand when the run finishes.
The run's final outcome was `done`.

This is abnormal because a background shell can outlive the decision to stop waiting, and its output is lost without any record beyond the note.
Suggested fix: stop or explicitly report the shell when the wait gives up, or make the shell-wait rule consistent between transcript-reported and Stop-payload-reported shells.

Provenance: task `daemon-wakeups`; note `webster-background-shell.md`.
