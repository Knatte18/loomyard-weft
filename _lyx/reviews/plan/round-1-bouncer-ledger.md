---
round: 1
ledger:
  - key: card3-drops-open-refusal-envelope-coverage
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding (F1): Card 3 makes every row of `TestOpenCmd_ModelAliasEngine` a successful open, leaving no CLI test of `Runner.Open` refusal reaching the error envelope.
The review's fix is a local test-row addition with the result fixed by the finding text.
