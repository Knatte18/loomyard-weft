---
round: 1
ledger:
  - key: interrupt-settle-no-run-to-resolve
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: webster-default-engine-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: attach-mismatch-skips-requires
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: r2-misstates-changed-default
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: instruction-files-no-scenario
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open.
Three are design gaps in D4, D7 and D8: a per-run engine read with no record to resolve from, an unbounded default-engine path in Webster, and an attach edge that skips `Requires`.
Two are local: R2's wording and a missing `InstructionFiles` acceptance scenario.
