---
round: 3
ledger:
  - key: dp-infeasible-card-over-budget
    rounds: [1]
    status: resolved
  - key: calibration-before-estimator-order
    rounds: [1]
    status: resolved
  - key: weights-circular-calibration-profile
    rounds: [1]
    status: resolved
  - key: decisions-4-5-model-mismatch
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: past-run-base-tree-two-phrasings
    rounds: [1]
    status: resolved
  - key: order-assertion-and-plan-change-unplaced
    rounds: [1]
    status: resolved
  - key: midrun-load-error-unbounded
    rounds: [1]
    status: resolved
  - key: legacy-state-fallback-unbounded
    rounds: [1]
    status: resolved
  - key: decision-10-lists-status-verb
    rounds: [1]
    status: resolved
  - key: decision-12-hash-and-repeat
    rounds: [1]
    status: resolved
  - key: cost-model-never-rewards-merging
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: calibration-before-profile-loader
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: threshold-unit-vs-scale-factor
    rounds: [2, 3]
    status: open
    class: decision
    severity: MEDIUM
  - key: rebaseline-without-recorded-partition
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: poor-fit-no-criterion
    rounds: [2]
    status: resolved
  - key: measured-weight-several-forks-or-none
    rounds: [2]
    status: resolved
  - key: invalid-parameter-undefined
    rounds: [2]
    status: resolved
  - key: rebaseline-card-naming-guard
    rounds: [2]
    status: resolved
  - key: unused-signals-and-package-entries
    rounds: [2]
    status: resolved
  - key: interview-stale-decision-number
    rounds: [2]
    status: resolved
  - key: mixed-partition-profile-name
    rounds: [3]
    status: open
    class: decision
    severity: LOW
  - key: notes-call-site-list
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
---

Round 3 raised seven findings, none BLOCKING and none `design`.
Six map to earlier keys, reopened in a narrower form: F1 to `threshold-unit-vs-scale-factor`, F2 to `decisions-4-5-model-mismatch`, F3 to `calibration-before-profile-loader`, F4 to `rebaseline-without-recorded-partition`, F6 to `cost-model-never-rewards-merging`.
F5 (`mixed-partition-profile-name`) and F7 (`notes-call-site-list`) are new.
The round-2 design findings are downgraded by the fresh review to wording and ordering issues: the per-fork term now exists and only its rationale attribution remains (F6), and the card order is a consistency matter (F3).
The six round-2 keys the round-3 review did not raise again are marked resolved.
