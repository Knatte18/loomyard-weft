---
round: 2
ledger:
  - key: transient-githubapi-doc-claims-rate-limit
    rounds: [1]
    status: resolved
  - key: fixed-column-wrapped-new-comments
    rounds: [1]
    status: resolved
  - key: burlerengine-out-of-card-gofmt-whitespace
    rounds: [1]
    status: resolved
  - key: istransient-url-error-parse-and-tls
    rounds: [1]
    status: resolved
  - key: parking-records-command-untested
    rounds: [1]
    status: resolved
  - key: no-attach-help-ordering
    rounds: [1]
    status: resolved
  - key: rundriverspawnandwait-doc-omits-resume
    rounds: [1]
    status: resolved
  - key: startup-test-doc-comment-space-dropped
    rounds: [1, 2]
    status: resolved
  - key: run-commitstatus-test-out-of-card-reformat
    rounds: [2]
    status: resolved
---

Round 1 resolved seven findings and left one open, a space that card 4's commit `3e5420f01` dropped from a doc comment in `internal/shuttleengine/startup_test.go:768`.
In round 2 the review raised it again as F1, and `cf86eab89` restored the space, which resolves it.

Round 2 also raised F2, a NIT calling card 1's reformat of `internal/shedengine/run_commitstatus_test.go` out-of-card churn.
The fixer withdrew it, and I confirmed that the merge-base file fails `gofmt -l` under go1.26.0.
The reformat is gofmt's own output, so F2 is resolved with no change.

The whole-range whitespace sweep at `cf86eab89` finds 11 pairs that differ only in whitespace, and all are gofmt re-alignments.
No finding is open.
