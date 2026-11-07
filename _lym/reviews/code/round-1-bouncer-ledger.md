---
round: 1
ledger:
  - key: stored-record-link-in-same-write-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: render-to-disk-copy-only-discussed-flag-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW test-coverage findings.
F1 (`stored-record-link-in-same-write-unpinned`): no test checks that a record stored through the board write is linked in the same write's README and design doc.
F2 (`render-to-disk-copy-only-discussed-flag-unpinned`): no test checks that `RenderToDisk` leaves the caller's entries flagged discussed.
Neither gates, and the loop converged in round 1.
