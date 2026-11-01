# pr-continue wakes the PR-Review wait at once

`lym loom pr-continue` only sets the continue flag in the run's pull-request record.
The PR-Review row's wait read that flag at its next wake, and between wakes it slept from one second up to a minute while nothing changed.
So the operator's ending of a review could wait up to a minute before the row closed the pull request and Finalize began.

The wait now also wakes on a change to the record file.
A change ends the sleep between two wakes early,
and the next wake reads the flag, polls the pull request's status at once as it already did for a newly seen flag, closes the pull request and ends Done.

## Changes

- `landingshed.PullRequestDeps` gains a required `RecordChanges` seam.
  It opens a watch on the record file and returns a change channel and the function that ends the watch.
  A nil channel leaves the wait on its timer.
- `PRReview.wait` opens the watch when the wait starts and ends it on the way out.
  Its sleep between wakes now goes through `pause`, which ends the sleep when the watch delivers a change.
  A change still waits out the rest of the one-second floor since the last wake began, so wakes stay at least a second apart.
- `loomcli` wires the seam to an `internal/fswatch` watch on `pull-request.json` through a new `watchFileChanges` helper.
  When no watch can be opened, the helper logs a Warn and returns a nil channel, so the wait falls back to its timer.
  Pawl's land-request watch, which carried the same forwarding loop, now uses this helper.
- The landingshed package doc and `PATTERN-daemon-wakeups`'s list of unbounded waits describe the event-driven wake.

## Design decisions

- The wake signal is the record file itself, not a separate wake file or socket.
  `pr-continue` stays unchanged and still writes only the flag, so the verb and the row share no new channel.
- The row's own writes to the record also fire the watch.
  Each such write costs at most one extra wake, which reads only local state and keeps the one-second floor.
  Filtering out the row's own writes would add state for no gain.
- `RecordChanges` is required like every other `PullRequestDeps` seam, so each fixture states its watch.
  Batten's and loom's wire-seam tables list it with the other landing seams that `wire()` leaves nil.

## Tests

`TestPRReview_Wait` gains a case where the first sleep writes the continue flag, signals a change and returns only when the wait cancels it.
The wait then sleeps the rest of the floor, closes the pull request and ends Done, and the watch is ended once.
A real-time guard in that fake fails the case, rather than letting it hang, if the change never ends the sleep.
