---
round: 1
ledger:
  - key: resume-finished-cancel-reads-running
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: undecodable-record-names-unrunnable-cancel
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resume-owner-refusal-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: contained-tests-lack-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four LOW findings, F1 to F4, in the order of the ledger keys.
None is gating, so the loop converged.
