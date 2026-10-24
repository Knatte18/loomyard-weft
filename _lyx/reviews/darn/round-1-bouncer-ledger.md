---
round: 1
ledger:
  - key: untuned-darn-defaults-migrate-to-loom
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: commit-discussion-snapshot-order-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: new-key-moves-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: configcli-doc-retired-key-example
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reroute-reviewed-code-budget-zero-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: snapshot-refusals-lack-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: redundant-perch-hard-cap-row
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, none carried from an earlier round.
The gating one is F1: a pre-sparse darn.yaml still holding darn's former defaults moves into loom.yaml as a tuned value, lowering loom's Code-Review hard cap and replacing its reviewer model.
F2 to F7 are test-coverage gaps, a stale doc example, a refusal without a way forward and one redundant test row.
