---
round: 2
ledger:
  - key: auto-accept-after-fix-gate-exhausted
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: acceptance-record-archive-lifecycle
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
  - key: empty-review-skip-tolerant-parse
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
  - key: budget-of-one-gate-exhaustion-criterion
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised three findings; round 2 reopens all three in narrower form and adds one.
The auto-accept hazard persists: after a fix gate is exhausted, the zero-findings skip in D4 runs no gate on the re-review, so a failed-gate target can reach the judge clean.
The archive lifecycle gap is now the fixer-died-or-timed-out exit.
The tolerant-parse gap is now the APPROVED review with a misnamed findings key and no prose blocks.
The new key is the missing acceptance criterion for the budget-of-1 gate-exhaustion halt.
