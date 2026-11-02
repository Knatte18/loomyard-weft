---
round: 1
ledger:
  - key: tier1-command-lacks-test-timeout
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding (F1): the `{test_timeout}` mechanism landed, but no tier1 command in this checkout uses it.
The hub's `gate.yaml` change is the parent's to record or perform.
