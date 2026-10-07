---
round: 2
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
    rounds: [1, 2]
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
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: calibration-before-profile-loader
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: threshold-unit-vs-scale-factor
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: rebaseline-without-recorded-partition
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: poor-fit-no-criterion
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: measured-weight-several-forks-or-none
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: invalid-parameter-undefined
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: rebaseline-card-naming-guard
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: unused-signals-and-package-entries
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: interview-stale-decision-number
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

The nine round-1 findings the round-2 review did not raise again are marked resolved, since the fresh review no longer finds them.
The one carried finding is the Decision 4 and Decision 5 model mismatch (F3), still open in a new form.
Round 2 adds ten keys, F1 and F2 to F11 less F3.
F1 (BLOCKING `design`) is the gating finding: Decision 5 has no per-fork term, so merging never pays.
F2 (MEDIUM `design`) is the second: the calibration report is ordered before the profile loader it needs.
