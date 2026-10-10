---
round: 3
ledger:
  - key: test-budget-rows-past-test-count
    rounds: [1]
    status: resolved
  - key: driver-bootstrap-stub-stale-launch-flow
    rounds: [2]
    status: resolved
  - key: smoke-stub-stale-skill-load-flow
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised the budget-row finding (LOW, consistency); it was not seen again and stays resolved.
Round 2 raised the BLOCKING driver-bootstrap stub finding; round 3 confirms the tmux tier passes, so it is resolved.
Round 3 raised F1 (LOW, consistency): the smoke driver-strand stub and its doc comment still describe the retired skill-load flow.
