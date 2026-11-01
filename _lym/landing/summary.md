# pr-continue wakes the PR-Review wait at once

`lym loom pr-continue` only writes a continue flag into the pull-request record, and the PR-Review wait read it on its next wake.
That wake could be up to a minute away, since the wait doubles its sleep from one second to a minute while nothing changes.
The wait now watches the record file and ends its sleep when the file changes, so the PR is closed and Finalize begins within about a second of the command.

## Design

- `landingshed.PullRequestDeps` gains a `RecordChanges` seam, which opens a watch on the record and returns a change channel with a stop function.
  `PRReview.wait` opens it once per call and `PRReview.pause` selects between the timer sleep and the channel.
  `checkWiring` rejects a missing `RecordChanges`.
- A change does not wake the row sooner than `prWaitFloor` after the last wake began, so no wake follows another by less than a second, and the wait's own record writes cannot spin it.
- A nil channel, as when no watch can be opened, leaves the wait on its timer exactly as before.
- `loomcli.watchFileChanges` is extracted from the land-request watch in `pawlwiring.go` and now serves both that watch and the PR-Review record watch.
  It logs a Warn naming the waiter when the watch cannot be opened, and merges changes that arrive while one is undelivered.
- The record watch is closed when the wait ends.
  The land-request watch still lives until the process ends.
- `envkit.LandingDeps` supplies a nil-channel `RecordChanges`, so existing landing tests keep their timer-only behaviour.
- `pattern/PATTERN-daemon-wakeups.md` lists the PR-Review wait among the unbounded waits, and the `landingshed` package doc describes the early wake.

## Tests

- `prreview_wait_test.go` covers a record change ending the sleep early, a change part-way through the floor resting only the remainder, a change after the floor waking at once, and the watch being stopped once.
- `watchfilechanges_test.go` covers `watchFileChanges` against a real directory, and that `pullRequestDeps` watches the run's pull-request record.

## To check by hand

- `watchFileChanges` in `internal/loomcli/landingdeps.go`, which both the PR-Review record watch and Beam-Await now share: a review flagged it as untested, and `watchfilechanges_test.go` was added afterwards.
  Confirm the tests pin a write to the watched file, the one buffered value, a sibling file's write delivering nothing, the nil channel for a missing directory, and `stop` being safe to call twice.
- Not run: a real PR-Review run where the reviewer agent calls `lym loom pr-continue`, checking that Finalize starts within about a second.
