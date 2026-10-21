---
round: 1
ledger:
  - key: pattern-durable-state-index-clause-dropped
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: pattern-push-both-sides-index-clause-dropped
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: gateslot-loadconfig-local-removal-for-ignored-file
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: wrapper-and-way-forward-wording-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: loadconfig-two-file-local-removal-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: layerpath-machine-owner-unpinned
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 opened six findings.
F1 and F2 are BLOCKING: the PATTERN.md edit dropped clauses from two index lines that no declared change covers.
F3 is a LOW correctness gap in `LoadConfig`'s local-removal step.
F4 to F6 are missing tests for the wrapper wording, the two-file local-removal clause and the machine layer owner.
