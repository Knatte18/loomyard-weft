---
round: 1
focus:
  - "internal/landingshed/prreview.go `pause`: settle whether the Sleep goroutine, the cancel on a change, the `<-slept` drain and the follow-up floor Sleep leave a path that leaks, blocks, or returns the wrong error on context cancellation, in both select branches."
  - "internal/landingshed/prreview.go `wait` and `wake`: settle whether a record change wakes the wait without resetting the doubling `interval`, and whether the wait's own record writes (status, head or continue-flag updates) can retrigger the watch so the wait wakes every floor instead of backing off to a minute while nothing else changes."
  - "internal/loomcli/landingdeps.go `RecordChanges` and `watchFileChanges`: settle whether the watched path (`shedrun.PullRequestRecord` with `SelfRunID`) is the file `lym loom pr-continue` writes, whether its directory exists when the watch opens, and whether the nil-channel fallback and the `stop` function are correct on every return."
  - "internal/loomcli/pawlwiring.go `openLandRequestWatch`: settle whether the extracted helper keeps Pawl's earlier behavior, including the watcher's lifetime and the log text and level."
  - "Scope against the decision record's design (`pr-continue` wakes the waiting row at once): settle whether the diff makes that change and nothing beyond it, and whether the `PullRequestDeps.RecordChanges` wiring is complete in every place the sibling seams are listed, including the test seam tables and `internal/testkit/envkit/envkit.go`."
  - "Test economy in internal/landingshed/prreview_wait_test.go: settle whether the new test covers the early wake and the floor behavior of `pause`, and whether any added test duplicates an existing wait test."
  - "Doc comments and the `pattern/PATTERN-daemon-wakeups.md` entry: settle whether the new and changed comments and the entry follow the repository's conventions and match what the code does, and whether the commit message claims hold against the diff."
---

Seed focus from the decision record's Goal and the range diff.
