---
round: 1
ledger:
  - key: beamup-warning-surfacing-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: runs-strand-precedence-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: status-strand-strands-arg-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overview-doc-stale-overview-slot
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all fixable by local corrections the finding text spells out.
F1 (`beamup-warning-surfacing-untested`) is the only gating one: it needs a notify seam and a table-driven test.
F2 and F3 are test-coverage gaps, and F4 is a stale `docs/overview.md`.
