---
round: 1
ledger:
  - key: role-opening-stencils-lacks-plan-fixer
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: plan-fixer-audit-exclusion-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: fix-plan-refusals-lack-reaching-tests
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: plan-fix-stencil-marker-friction-tables
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: spawn-subtest-asserts-static-stencil-prose
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: conflict-record-duplicate-parse-unused-cause
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stale-verb-lists-in-comments
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, none BLOCKING.
The one gating finding is the untested exclusion of plan fixer sessions from the run audit.
Every fix is a local correction its finding text already specifies.
