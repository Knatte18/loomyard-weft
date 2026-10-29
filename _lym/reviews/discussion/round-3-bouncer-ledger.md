---
round: 3
ledger:
  - key: mode-check-research-deferred
    rounds: [1]
    status: resolved
  - key: operator-stop-copy-clamped-5-lines
    rounds: [1]
    status: resolved
  - key: version-risk-no-trigger
    rounds: [1]
    status: resolved
  - key: ring-wake-operator-copy-unstated
    rounds: [1, 2]
    status: resolved
  - key: ring-answer-path-early-exit-release
    rounds: [1, 2]
    status: resolved
  - key: ack-journaled-before-confirmation
    rounds: [1, 2]
    status: resolved
  - key: only-overstates-ring-answered
    rounds: [1]
    status: resolved
  - key: d5-journal-failure-no-outcome
    rounds: [2]
    status: resolved
  - key: ack-journal-failure-echo-rule
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised seven findings, round 2 four, round 3 one.
Every key open after round 2 was not raised again in round 3 and is resolved, including the ring wake's operator copy, the ring answer path's early exits and the journaling-before-confirmation order.

Round 3 raised one new LOW design key: D6 has no echo rule for an acknowledgement whose journal write fails, on a ring or on a claim that hands out an echo.
Its fix adds a rule to D6, so it is `design-wrong` and the new text needs a fresh look.
