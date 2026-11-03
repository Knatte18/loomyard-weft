---
round: 1
ledger:
  - key: status-read-cannot-tell-task-run-from-beam
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: status-read-conflicts-with-run-loop-item-4
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: anchor-description-incomplete
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: only-beam-create-starts-a-beam-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings, all in the new "Making a beam" section of `contracts/orchrole/orch-role.md`.
The gating one is the status-read test, which reads batten's own producer and so cannot separate a task run from a beam.
The other three are wording corrections: the `--anchor` description, the tie to run-loop item 4, and the starter claim.
