---
round: 3
ledger:
  - key: darn-write-append-unbounded
    rounds: [1, 2]
    status: resolved
  - key: darn-alignment-judged-against-writer-record
    rounds: [1, 2]
    status: resolved
  - key: unrecorded-correction-reverted
    rounds: [1, 2]
    status: resolved
  - key: commit-race-false-premise
    rounds: [1]
    status: resolved
  - key: constraints-carry-bookkeeping
    rounds: [1]
    status: resolved
  - key: d10-no-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: carried-over-correction-no-pr-align-rule
    rounds: [2]
    status: resolved
  - key: d7-bouncer-premise-false
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: interview-contradicts-current-d5
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 3 cleared every round 2 key: the review's Rejected section finds the dropped-entry checks, the PR-Align route for an unrecorded correction and the `pr-gate` scope-deviation definition sound.
Two new findings were raised.
F1 is D7's false premise about the Bouncer's CONTINUE and circling rules, with an over-stated acceptance line.
F2 is a support-log Interview note that contradicts the current D5.
