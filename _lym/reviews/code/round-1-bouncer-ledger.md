---
round: 1
ledger:
  - key: newfinalize-flaky-sink-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: rerun-guards-and-outcomes-lack-rows
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: package-failure-outside-test-reruns-whole-call
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: report-producer-keeps-literals-parser-names
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: remaining-coverage-gaps
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gate-slot-background-silent-on-rerun
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: call-doc-omits-github-merge
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings.
Two are gating: the `NewFinalize` flaky-sink wiring has no test, and the rerun guards and outcomes have no rows.
The rest are a deterministic package failure that reruns the whole call, producer literals that duplicate parser constants,
remaining coverage gaps, a stale gate-slot background, and a Call doc that omits the GitHub merge.
