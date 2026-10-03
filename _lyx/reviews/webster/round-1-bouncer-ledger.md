---
round: 1
ledger:
  - key: batten-shutdown-halts-reentered-teardown
    rounds: [1]
    status: resolved
  - key: record-commit-trailer-before-empty-check
    rounds: [1]
    status: resolved
  - key: remote-skipped-reason-doc-one-cause
    rounds: [1]
    status: resolved
  - key: new-comments-fixed-column-wrapped
    rounds: [1]
    status: open
  - key: session-ended-doc-overstates
    rounds: [1]
    status: resolved
---

Round 1 reviewed the whole branch over `main`, since no rework round exists.
Fixes for the batten re-entry halt, the record commit's trailer lookup, the `RemoteSkippedReason` doc and the `SessionResult.Ended` doc are in place, with integration tests where behavior changed.
One finding is still open: the comment reflow is incomplete.
Two comments added on this branch still break mid-sentence at a fixed column, at `internal/reedengine/overlay.go:250-251` (card 6) and `internal/reedengine/state.go:6-7` (card 7).
