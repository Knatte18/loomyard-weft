---
round: 2
ledger:
  - key: test-budget-rows-past-test-count
    rounds: [1]
    status: resolved
  - key: driver-bootstrap-stub-stale-launch-flow
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
---

Round 1 raised the budget-row finding (LOW, consistency); round 2 did not see it again, so it is resolved.
Round 2 raised F1 (BLOCKING, scope): the driver-bootstrap integration stub still follows the skill-load launch flow that card 17 removed, which fails the Publish verify.
