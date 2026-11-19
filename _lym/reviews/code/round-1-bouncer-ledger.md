---
round: 1
ledger:
  - key: code-ask-misses-rounds-own-review
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: parent-agreement-not-bound-to-rulings
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: parent-gate-keys-open-to-local-layer
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: drift-key-name-differs-from-entry
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: judge-parent-wait-outlives-judge
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: production-seat-construction-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: judge-table-agreement-entry-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: contained-tests-lack-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gate-rows-missing-repost-resolve-race
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: each-round-seat-round-one-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: judge-seat-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-answer-refusal-names-decision-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parentask-doc-stale-for-drift-judge
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: foreign-ask-latest-rerolls-discussion-ask
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised fourteen findings, none carried from an earlier round (F1 to F14 in review order).
Three are BLOCKING: the Code-Review ask timing (F1), the agreement's binding to the rulings (F2), and the Guard category for the new keys (F3, the parent's call).
The drift key name (F4) is likewise the parent's call.
The rest are the uncapped judge wait (F5), test gaps (F6, F7, F9, F10, F11), housekeeping (F8, F12, F13) and the shared ask record's latest lookup (F14).
