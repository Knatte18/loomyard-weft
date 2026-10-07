---
round: 1
ledger:
  - key: test-budget-not-raised
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: template-comment-names-ttl-vocabulary
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first round, so there is no earlier ledger.

- `test-budget-not-raised` (F1): two cards each add one top-level test function to a package already at its test-budget row, and neither lists the budget file among its targets.
- `template-comment-names-ttl-vocabulary` (F2): the `template.yaml` comment in a `shuttleengine` file spells the accepted TTL values, which the decision record confines to the engine's refusal.
