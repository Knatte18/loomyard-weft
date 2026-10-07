---
round: 1
ledger:
  - key: card2-formbatches-first-init-caller-missing
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card11-blankquoted-passes-double-quoted-substitution
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: card10-unbounded-entry-points-skip-fabric-reference
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card10-tests-need-verify-field-plankit-lacks
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: cards5-6-plan-commit-read-planned-twice
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card2-repeats-card1-over-budget-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card8-table-restates-refscanner-matches-coverage
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card10-asserts-one-behavior-at-three-test-sites
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card3-end-to-end-base-tree-unspecified
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: card11-stale-head-moved-refusal-row
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card5-impactsummary-restates-intent
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eleven findings, F1 to F11 in the order above.
The only `design` finding is F5, LOW, so the loop converges; the other findings are still open for the fixer.
