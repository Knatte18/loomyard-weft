---
round: 1
ledger:
  - key: gate-slot-holders-unlisted
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: unreachable-production-functions-between-cards
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: template-line-breaks-config-helper
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: bound-key-doc-text-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: reach-range-cap-no-reaching-input
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: mutate-budget-branches-unspecified
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: open-absent-map-directory-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reach-turn-halts-gate-on-record-failure
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: prime-report-wrong-reason
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: triage-prime-departs-from-no-parent-rule
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: card-13-site-struct-unlisted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round and has no prior ledger.
Eleven findings are open.
F1 (BLOCKING) and F2 (MEDIUM) are design-class and gate the loop.
F1's fix adds edit targets and doc text to cards 6 and 14.
F2's fix folds cards 8, 11 and 13 into their callers and moves card 3's index write, which renumbers cards.
The rest are local corrections to card text, targets and test rows, bar F6, F8 and F10, which change a stated behavior or decision.
