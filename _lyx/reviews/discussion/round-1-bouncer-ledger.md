---
round: 1
ledger:
  - key: d3-phase-entered-at-moves
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d7-seed-record-breaks-seed-contract
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d7-no-notice-bound
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d8-reed-single-window-enumeration
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d8-remain-on-exit-and-duplicate-bound
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d5-stop-report-and-since-untied
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-held-notice-vs-d4-done-return
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-quiet-bound-incomplete
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-api-error-text-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-arms-pause-seam-probe-rate
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-exempt-stuck-constraint-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d2-not-needed-by-d1
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d7-one-channel-per-event
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: a1-a8-not-assumptions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised 14 findings against the discussion pair, with no earlier ledger.
The gating cluster is mechanism-level: D3's request-covered test, D7's watched-marker and its notice bound, D8's second window against reed's single-window addressing, and D4 to D6's notice ordering, bounds and probe rate.
The consistency findings are record hygiene (D8 window closing, Constraints versus D4, D2, D7's channel claim, the two assumptions).
