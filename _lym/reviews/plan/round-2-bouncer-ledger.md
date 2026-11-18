---
round: 2
ledger:
  - key: gate-slot-holders-unlisted
    rounds: [1]
    status: resolved
  - key: unreachable-production-functions-between-cards
    rounds: [1]
    status: resolved
  - key: template-line-breaks-config-helper
    rounds: [1]
    status: resolved
  - key: bound-key-doc-text-stale
    rounds: [1]
    status: resolved
  - key: reach-range-cap-no-reaching-input
    rounds: [1]
    status: resolved
  - key: mutate-budget-branches-unspecified
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: open-absent-map-directory-unstated
    rounds: [1]
    status: resolved
  - key: reach-turn-halts-gate-on-record-failure
    rounds: [1]
    status: resolved
  - key: prime-report-wrong-reason
    rounds: [1]
    status: resolved
  - key: triage-prime-departs-from-no-parent-rule
    rounds: [1]
    status: resolved
  - key: card-13-site-struct-unlisted
    rounds: [1]
    status: resolved
---

Round 1 raised eleven findings; round 2 resolved ten.
The one still open is `mutate-budget-branches-unspecified` (F12 carrying F6), now LOW and consistency-class.
Card 13's `mutation_budget_min` row needs its hub config to raise `redundancy_build_timeout_min`, because the build bound and the budget share one `minuteUnit`.
