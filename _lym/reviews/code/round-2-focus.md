---
round: 2
focus:
  - "internal/loomcli/landingdeps.go `watchFileChanges` and the `RecordChanges` seam built in `pullRequestDeps`: does a test write to the watched file and read the channel, covering a sibling file's write, merged bursts, a missing directory and stop, and does the watched path equal `shedrun.PullRequestRecord(l, shedrun.SelfRunID)`?"
  - "internal/landingshed/prreview_wait_test.go `TestPRReview_Wait`: after a change signal, is the floor rest pinned for a clock already partly past the floor and for one fully past it, so that a stale `wokeAt` fails a row?"
---

Both entries target the tests added since round 1; judge them as new text.
