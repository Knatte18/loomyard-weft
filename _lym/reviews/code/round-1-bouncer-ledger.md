---
round: 1
ledger:
  - key: joint-commit-groups-do-not-chain
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: commit-order-vs-group-rule
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overlay-subtest-vacuous-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fix-stencil-calls-source-rules-commit-per-fix
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: finding-doc-location-meaning
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fix-site-location-vs-carried-path
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all with fix text that names the exact change.
F1 (chaining of groups) is the only gating one.
