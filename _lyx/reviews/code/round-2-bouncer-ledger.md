---
round: 2
ledger:
  - key: role-opening-stencils-lacks-plan-fixer
    rounds: [1]
    status: resolved
  - key: plan-fixer-audit-exclusion-untested
    rounds: [1]
    status: resolved
  - key: fix-plan-refusals-lack-reaching-tests
    rounds: [1]
    status: resolved
  - key: plan-fix-stencil-marker-friction-tables
    rounds: [1]
    status: resolved
  - key: spawn-subtest-asserts-static-stencil-prose
    rounds: [1]
    status: resolved
  - key: conflict-record-duplicate-parse-unused-cause
    rounds: [1]
    status: resolved
  - key: stale-verb-lists-in-comments
    rounds: [1]
    status: resolved
  - key: restore-plan-operator-audience-named-by-role-prompt
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, none BLOCKING; the fixes landed and round 2 did not re-raise any of them.
Round 2 raised one LOW consistency finding: `restore-plan` should be role-audience because the plan fixer's prompt names it.
Its fix is a local annotation change the finding text already specifies.
