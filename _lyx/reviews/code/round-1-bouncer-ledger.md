---
round: 1
ledger:
  - key: start-go-stale-marker-dir-comments
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: dobbygeometry-doc-stale-marker-paths
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tidy-report-path-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings: F1 and F2 are stale marker-directory wording in doc comments, F3 is a missing assertion on the tidy report path.
None gates, so the loop converged in the first round.
