---
round: 1
ledger:
  - key: card2-impactsummary-caller-set
    rounds: [1]
    status: resolved
  - key: card3-impactsummary-restates-intent
    rounds: [1]
    status: resolved
  - key: card2-reader-placement-mergestate
    rounds: [1]
    status: resolved
  - key: card3-conditional-test-rename
    rounds: [1]
    status: resolved
---

Round 1 raised four findings, and the fixer resolved all four in the same round.
Card 2's `ImpactSummary` (F1) and card 3's (F2) now state concrete blast-radius conclusions.
Card 2's extracted four-probe reader (F3) moved into `mergestate.go` beside `foreignMergeStatePresent`.
Card 3's test rename (F4) is no longer conditional.
No finding is open, and the plan is approved at round 1.
