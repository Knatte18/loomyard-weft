---
round: 5
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
    status: resolved
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
    status: resolved
  - key: d4-since-not-lower-bound
    rounds: [3, 4, 5]
    status: open
    class: consistency
    severity: LOW
  - key: d3-baseline-stat-before-read
    rounds: [3, 4]
    status: resolved
  - key: d7-chained-set-option-no-target
    rounds: [3, 4]
    status: resolved
  - key: d4-not-queued-retry-unbounded
    rounds: [3, 4]
    status: resolved
  - key: batten-refusal-spec-row-no-home
    rounds: [4]
    status: resolved
  - key: d6-marker-not-tied-to-driver-parent
    rounds: [5]
    status: open
    class: decision
    severity: MEDIUM
  - key: d7-recorded-pane-id-trust
    rounds: [5]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-quiet-fallback-stale-clock
    rounds: [5]
    status: open
    class: consistency
    severity: LOW
  - key: d7-window-verb-refusals-incomplete
    rounds: [5]
    status: open
    class: decision
    severity: LOW
  - key: open-risks-repeats-bounds
    rounds: [5]
    status: open
    class: consistency
    severity: NIT
---

Findings per round: 14, 10, 9, 7, 6.
Highest severity has been MEDIUM since round 3, and gating findings fell from nine to one.

Round 5 mapping:
- F1 is new as `d6-marker-not-tied-to-driver-parent` (`decision`, MEDIUM): the watched marker tests where batten runs, not the driver's parent.
  The earlier relay key is resolved, since the review accepts the 10-minute notice as the bound.
- F2 is new as `d7-recorded-pane-id-trust` (`design`, MEDIUM): the generation check fails open and pane ids are server-wide.
  It is the only gating finding.
- F3 continues `d4-since-not-lower-bound` (LOW): the carrier of `since` across calls is unstated.
- F4 is new as `d5-quiet-fallback-stale-clock` (LOW): the quiet fallback uses the history time D4 calls stale.
- F5 is new as `d7-window-verb-refusals-incomplete` (LOW): the refusal count is pinned and ways forward are missing or misleading.
  The earlier spec-row key is resolved, since the review verifies the out-of-scope claim.
- F6 is new as `open-risks-repeats-bounds` (NIT).

Resolved this round: `d8-reed-single-window-enumeration`, `d3-baseline-stat-before-read`, `d7-chained-set-option-no-target`, `d4-not-queued-retry-unbounded`, `batten-refusal-spec-row-no-home` and `awaiting-parent-notice-relay-unbounded` are no longer raised, or the review verified them as not defects.
