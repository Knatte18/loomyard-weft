---
round: 1
ledger:
  - key: lym-batching-override-bullet-names-no-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: plan-edits-step-4-sub-steps-unnumbered
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW consistency findings in `internal/heddleengine/shipped.json`.
F1: the override bullet in `lym-batching` says "the rule above" and so covers only the preceding bullet.
F2: step 4 of `lym-plan-edits-before-gate` uses unordered sub-bullets where order matters.
Both fixes are local corrections whose text the findings already fix.
