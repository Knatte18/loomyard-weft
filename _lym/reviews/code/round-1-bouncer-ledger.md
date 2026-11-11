---
round: 1
ledger:
  - key: before-build-dirty-skip-counts-against-max
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: head-moved-check-over-linked-files-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-build-map-format-marker-unasserted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: nested-package-rule-branches-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: before-build-dirty-check-not-shown-fresh
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failure-paths-and-unresolved-verdict-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: stamp-digests-and-gosumhash-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: nested-package-rule-departs-from-impactset
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings: F1 (--max accounting), F2 to F5 (coverage gaps in the testmap tests) and F6 to F8 (LOW).
Keys map to findings in order, F1 to F8.
Every fix is a local correction whose result the finding text states.
