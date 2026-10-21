---
round: 1
ledger:
  - key: triage-section-names-retired-verify-suite
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: guard-row-sets-retired-publish-verify
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fixtures-write-retired-go-parallel
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: decodetier-accepts-negative-covers
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: timeout-halt-lacks-role-clause
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: decodetiers-discards-atoi-error
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: redundant-single-slot-subtest
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, none gating: one MEDIUM consistency, three LOW consistency, one LOW design, one NIT design and one NIT scope.
All fixes are local corrections the finding text already specifies.
