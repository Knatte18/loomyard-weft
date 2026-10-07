---
round: 1
ledger:
  - key: dp-infeasible-card-over-budget
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: calibration-before-estimator-order
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: weights-circular-calibration-profile
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: decisions-4-5-model-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: past-run-base-tree-two-phrasings
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: order-assertion-and-plan-change-unplaced
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: midrun-load-error-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: legacy-state-fallback-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: decision-10-lists-status-verb
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: decision-12-hash-and-repeat
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised ten findings, F1 to F10, mapped in order to the keys above.
Three are gating `design` findings: the infeasible DP for a card over budget alone (BLOCKING), the card order that puts the calibration report before the estimator, and the circular weights between report and template profile.
The rest are consistency or decision gaps in the cost model, base-tree rule, order assertion, load-error bound and legacy fallback, plus one LOW and one NIT.
