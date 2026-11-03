---
round: 1
ledger:
  - key: seed-discussed-record-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: commit-status-and-seed-wiring-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: glossary-motif-half-updated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: finalize-less-child-batten-handling
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all open.
The MEDIUM design finding is missing test rows for `seedDiscussedRecord`'s overwrite, missing-record and failure paths.
The LOW findings are unpinned wiring of the commit-status and seed seams, a glossary that still treats motif as reserved, and no batten handling or test for a motif child that ends without Finalize.
