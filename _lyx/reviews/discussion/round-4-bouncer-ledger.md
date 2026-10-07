---
round: 4
ledger:
  - key: d3-phase-entered-at-moves
    rounds: [1]
    status: resolved
  - key: d7-seed-record-breaks-seed-contract
    rounds: [1]
    status: resolved
  - key: d7-no-notice-bound
    rounds: [1, 2, 3]
    status: resolved
  - key: d8-reed-single-window-enumeration
    rounds: [1, 2, 3, 4]
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
    status: resolved
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
    status: resolved
  - key: done-notice-report-path-dead
    rounds: [2]
    status: resolved
  - key: d2-older-than-cycle-start-clause
    rounds: [2]
    status: resolved
  - key: awaiting-parent-notice-relay-unbounded
    rounds: [3, 4]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-since-not-lower-bound
    rounds: [3, 4]
    status: open
    class: consistency
    severity: LOW
  - key: d3-baseline-stat-before-read
    rounds: [3, 4]
    status: open
    class: design
    severity: LOW
  - key: d7-chained-set-option-no-target
    rounds: [3, 4]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-not-queued-retry-unbounded
    rounds: [3, 4]
    status: open
    class: decision
    severity: LOW
  - key: batten-refusal-spec-row-no-home
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised 14 findings, round 2 ten, round 3 nine, round 4 seven; the highest severity stayed at MEDIUM since round 3.

Round 4 mapping:
- F1 continues `awaiting-parent-notice-relay-unbounded` at MEDIUM: the report-only failure record names the hole without bounding it, and the suppression is not tied to the notice's destination.
  It also absorbs the destination remainder of `d7-no-notice-bound`, now resolved as its own key.
- F2 continues `d7-chained-set-option-no-target` at MEDIUM: the targeting is right, but the shared-name failure premise is false.
- F3 continues `d8-reed-single-window-enumeration` at LOW: dead Selvage corpse and the pane-generation read.
- F5 continues `d4-since-not-lower-bound` as a LOW consistency item between D4 and D5.
- F6 continues `d3-baseline-stat-before-read` at LOW: the baseline ordering and coarse timestamps.
- F7 continues `d4-not-queued-retry-unbounded` at LOW: log bound and end of two retries.
- F4 is new at LOW: the refusal constraint names a spec row with no home.

Resolved this round: `d4-arms-pause-seam-probe-rate` and `history-entry-cannot-name-change` are no longer raised.
