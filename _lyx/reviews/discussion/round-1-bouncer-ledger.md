---
round: 1
ledger:
  - key: skill-listing-missing-classifies-all-unknown
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: load-turn-timeout-no-confirmation-source
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: reload-step-encoding-and-restart-durability
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: decision-13-unverified-premise-stated-as-closed
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: retry-turn-outcomes-partly-stated
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: decision-8-removed-guard-passes-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: decision-14-rejected-alternative-re-argued
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first round, so there is no prior ledger.
The review raised seven findings, F1 to F7, mapped in order to the keys above.
None is class `design` and none is BLOCKING, so the loop converges at this round.
