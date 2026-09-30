---
round: 1
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
    rounds: [1]
    status: open
---

Round 1's review raised seven findings, F1 to F7, and the fix round landed one commit for each.
I checked all seven against the diff at `38cb96779`, and all are resolved.
The plan's full verify block passes, and gofmt is clean.

One finding stays open.
Card 4's commit `3e5420f01` stripped a space from an untouched doc comment in `internal/shuttleengine/startup_test.go:768`, leaving `...NeverReadypins`.
It is the same kind of stray edit as F3, in the same commit, but gofmt cannot see it because it sits in a comment.
I compared every changed line pair across the range with whitespace ignored, and nothing else of this kind is left.
