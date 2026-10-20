---
round: 1
ledger:
  - key: dry-run-report-contradicts-d4
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: runcmd-cluster-fan-refusal-loses-reach
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: burler-yaml-text-sweep-misses-members
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: resolvefan-refusal-prefix-unsettled
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: configengine-doc-missing-pernamemaps
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: per-name-value-shape-ignores-path
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: partial-fix-of-failing-fans-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: set-overwrites-non-mapping-per-name-value
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: reconcile-non-mapping-refusal-no-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: retired-burler-refusals-no-next-step
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: setvalues-null-at-p-branch-no-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: printresolved-lenses-subtest-repeats-pin
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: loom-rows-empty-dirs-wording
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: hub-walk-log-omits-keptextra
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so the ledger carries only its own 14 findings (F1 to F14 in the review).
None is gating: the design findings (partial-fix way forward, `--set` over a non-mapping) are LOW, and the one MEDIUM is a decision-class departure from D4.
