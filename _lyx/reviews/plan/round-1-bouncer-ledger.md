---
round: 1
ledger:
  - key: null-per-perch-budget-breaks-config-set
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: eligible-stop-hold-drops-on-arbiter-circling-decision
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: reviewbudget-loaded-config-test-at-zero
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: zero-to-null-migration-skips-sparse-worktree-files
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: routing-projection-reports-engine-default-for-zero-round
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: darn-unparseable-document-refusal-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings.
Two are BLOCKING: null per-perch template leaves get a `!!null` tag when set through `lyx config --set` (card 3), and the eligible-stop hold ends early once the arbiter records its own circling decision (cards 7 and 8).
Four LOW findings concern `reviewBudget`'s loaded-config test, the 0-to-null migration scope, the routing projection's inherited budget for a zero-round perch, and an unreachable refusal in darn's validation.
