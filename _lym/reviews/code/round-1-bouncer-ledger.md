---
round: 1
ledger:
  - key: pattern-test-speed-background-sequential-tail
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: report-steps-share-last-tier-process-cap
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: guard-foreign-wiring-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: last-tier-start-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: gateslot-doc-sequential-tail
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: stopped-chain-row-tests-nothing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: halted-call-with-reports-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review; no earlier ledger exists.
Two documents still describe the report steps as running after the tiers (`pattern/PATTERN-test-speed.md`, `internal/gateslot/doc.go`).
One design defect lets the redundancy step's marked processes count toward the last tier's process cap.
Four test gaps remain: the guard's foreign-set wiring, the chain's start index, a halted call with reports, and a stopped-chain row that cancels nothing.
