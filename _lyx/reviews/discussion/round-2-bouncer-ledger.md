---
round: 2
ledger:
  - key: d3-phase-entered-at-moves
    rounds: [1]
    status: resolved
  - key: d7-seed-record-breaks-seed-contract
    rounds: [1]
    status: resolved
  - key: d7-no-notice-bound
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-reed-single-window-enumeration
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-remain-on-exit-and-duplicate-bound
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
  - key: d5-stop-report-and-since-untied
    rounds: [1]
    status: resolved
  - key: d5-held-notice-vs-d4-done-return
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d6-quiet-bound-incomplete
    rounds: [1]
    status: resolved
  - key: d6-api-error-text-unbounded
    rounds: [1]
    status: resolved
  - key: d4-arms-pause-seam-probe-rate
    rounds: [1, 2]
    status: open
    class: decision
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
    status: open
    class: design
    severity: BLOCKING
  - key: status-stat-failure-waits-forever
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: history-entry-cannot-name-change
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: done-notice-report-path-dead
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-older-than-cycle-start-clause
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's findings are mostly gone; round 2 raised 10 fresh ones, none a plain repeat.
Round 1 keys the round 2 review no longer raises are resolved: the `PhaseEnteredAt` test, the seed-record contract, the stop-report and `since` tie, the quiet-bound and API-error text, the exempt-`Stuck` constraint mismatch, D2's necessity, the one-channel claim and the two assumptions.

Four round 1 keys continue, now on a narrower point:

- `d7-no-notice-bound` continues as F3: the watched marker proves a live batten, not a notice destination.
- `d8-reed-single-window-enumeration` continues as F4: no source for the window id and an incomplete window-targeted op set.
- `d8-remain-on-exit-and-duplicate-bound` continues as F5: the batten window's lifecycle is partly open.
- `d5-held-notice-vs-d4-done-return` continues as F2: "sent before the call returns" against the 3-minute report wait.
- `d4-arms-pause-seam-probe-rate` continues as F8: the cadence of each read is not stated.

New in round 2: F1 (an awaiting stop with no `parent_notice` is announced nowhere, BLOCKING), F6 (a failed status stat waits forever), F7 (the history entry cannot name the change), F9 (a done notice's report path is dead on arrival) and F10 (D2's "older than the cycle start" clause).
