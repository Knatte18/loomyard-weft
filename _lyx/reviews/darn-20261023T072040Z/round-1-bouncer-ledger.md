---
round: 1
ledger:
  - key: withdrawn-resume-line-lost-on-failed-reread
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: no-test-post-withdraw-reread-failure
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: no-test-unconfirmed-outcome
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: no-test-withdrawn-resume-line-with-inflight-compact
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, F1 to F4 in that order.
F1 is the one gating finding: after a successful withdraw, a failed re-read drops the withdrawn resume line, so its pause-record mark is never cleared.
F2 to F4 are missing tests for that branch, the unconfirmed outcome and the in-flight path.
