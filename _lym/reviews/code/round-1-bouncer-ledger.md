---
round: 1
ledger:
  - key: judge-never-reads-charter
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: missing-board-entry-refusal-untested
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: seed-brief-failure-swallowed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: copies-tier-not-derived
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: crucible-carry-over-incomplete
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: fixer-rules-in-rubric-fixer-never-reads
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: recipe-seed-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: preflight-tolerates-absent-gate-yaml
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: preflight-test-skips-template-keys
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: removed-checks-no-loosened-check-trailer
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: default-sandbox-layout-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review of the range; no key recurs.
The two BLOCKING findings are the judge's missing access to the charter and the untested missing-board-entry refusal.
The seed-brief failure (F3) and the absent-gate.yaml preflight (F8) both break the rule that refusals come before any agent spends tokens.
Mapping: F1 to F11 in order of the keys above.
