---
round: 1
ledger:
  - key: depends-note-detail-and-precedence-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: child-output-append-edge-cases-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: lone-newline-stdout-writes-bare-prefixed-line
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings: two test-coverage gaps (F1, F2) and one doc-versus-behavior mismatch in `appendLoopChildOutput` (F3).
Each fix is a local correction its finding text already fixes.
No prior ledger exists.
