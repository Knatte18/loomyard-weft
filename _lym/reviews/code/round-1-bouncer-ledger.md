---
round: 1
ledger:
  - key: seed-precision-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: failure-branches-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: parent-rules-block-asserts-stencil-prose
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parent-coverage-shape-test-duplicate
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parent-copy-path-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: wiring-header-overstates-read-time
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six non-gating findings: two test-coverage gaps (F1, F2), three test-economy items (F3, F4, F5) and one header wording nit (F6).
Each fix is a local correction the review text already spells out.
