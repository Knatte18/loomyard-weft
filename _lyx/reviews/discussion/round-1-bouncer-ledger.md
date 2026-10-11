---
round: 1
ledger:
  - key: d5-freshness-rule-rejects-genuine-records
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d2-trust-claim-wrong-guard
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-ignores-rebase-retry-triggers
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-lock-refused-retry-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-land-failure-episode-undefined
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: goal-wake-list-disagrees-with-scope
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d6-cannot-lock-ref-not-always-a-race
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d1-dirty-prime-wait-when-skipped
    rounds: [1]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised eight findings, all still open.
The core problems are in D5 (a freshness rule that rejects every real failure record, and an unbounded retry that suppresses the notice), D2 (trust anchored on the wrong guard), and D6 (it ignores the existing rebase classification of a remote refusal).
D4's failure-episode key is undefined against its own way forward.
The remaining three are LOW wording and ambiguity items.
