---
round: 1
ledger:
  - key: card4-resume-edge-unbounded
    rounds: [1]
    status: resolved
  - key: prompt-suggestion-residual-wrong-card
    rounds: [1]
    status: resolved
  - key: card7-impactsummary-hides-config-load-break
    rounds: [1]
    status: resolved
  - key: card13-stop-rule-drops-resume-fallback
    rounds: [1]
    status: resolved
  - key: card13-soft-cycle-fires-before-setup-turn
    rounds: [1]
    status: resolved
  - key: card11-impactsummary-restates-intent
    rounds: [1]
    status: resolved
  - key: card12-starter-seam-implementers-undercounted
    rounds: [1]
    status: resolved
---

Round 1 raised seven findings.
Two were MEDIUM:
card 4's `--resume` edge stated no bound,
and the orchengine prompt-suggestion residual was assigned to card 13 instead of card 3, the card that changes the behaviour.
Five were LOW or NIT:
weak ImpactSummaries on cards 7, 11 and 12,
card 13's stop rule not naming the decision record's resume fallback,
and a timing race in the soft-cycle smoke test.
The fixer fixed all seven in the plan, and the cards now carry each fix.
No finding is open, and the plan was approved at round 1.
