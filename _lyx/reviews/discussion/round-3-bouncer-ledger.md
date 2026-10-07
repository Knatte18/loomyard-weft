---
round: 3
ledger:
  - key: d3-phase-entered-at-moves
    rounds: [1]
    status: resolved
  - key: d7-seed-record-breaks-seed-contract
    rounds: [1]
    status: resolved
  - key: d7-no-notice-bound
    rounds: [1, 2, 3]
    status: open
    class: decision
    severity: LOW
  - key: d8-reed-single-window-enumeration
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: LOW
  - key: d8-remain-on-exit-and-duplicate-bound
    rounds: [1, 2]
    status: resolved
  - key: d5-stop-report-and-since-untied
    rounds: [1]
    status: resolved
  - key: d5-held-notice-vs-d4-done-return
    rounds: [1, 2]
    status: resolved
  - key: d6-quiet-bound-incomplete
    rounds: [1]
    status: resolved
  - key: d6-api-error-text-unbounded
    rounds: [1]
    status: resolved
  - key: d4-arms-pause-seam-probe-rate
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: LOW
  - key: d4-exempt-stuck-constraint-mismatch
    rounds: [1]
    status: resolved
  - key: d2-not-needed-by-d1
    rounds: [1]
    status: resolved
  - key: d7-one-channel-per-event
    rounds: [1]
    status: resolved
  - key: a1-a8-not-assumptions
    rounds: [1]
    status: resolved
  - key: awaiting-no-parent-notice-unannounced
    rounds: [2]
    status: resolved
  - key: status-stat-failure-waits-forever
    rounds: [2]
    status: resolved
  - key: history-entry-cannot-name-change
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: done-notice-report-path-dead
    rounds: [2]
    status: resolved
  - key: d2-older-than-cycle-start-clause
    rounds: [2]
    status: resolved
  - key: awaiting-parent-notice-relay-unbounded
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-since-not-lower-bound
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-baseline-stat-before-read
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-chained-set-option-no-target
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-not-queued-retry-unbounded
    rounds: [3]
    status: open
    class: design
    severity: LOW
---

Round 1 raised 14 findings, round 2 ten, round 3 nine; the highest severity fell from BLOCKING to MEDIUM.

Round 3 mapping:
- F1 is the narrowed remainder of the round 2 awaiting-stop key: the `parent_notice` relay exists now, but a dead driver or failed send leaves the stop unannounced with no bound.
- F2, F3 and F4 are new defects: the `since` time source, the modification-time baseline taken after the status read, and the chained `set-option` that lands on the strands' window.
- F5, F7, F8 and F9 continue the window-enumeration, notice-destination, read-cadence and history-text keys at LOW.
- F6 is new at LOW: the not-queued retry never stops.

Round 2 keys the round 3 review no longer raises are resolved: the failed-stat wait, the done-notice report path and D2's cycle-start clause, plus the round 2 forms of the 3-minute wait and the batten window lifecycle, which round 3 replaced with the narrower F2 and F4 keys.
