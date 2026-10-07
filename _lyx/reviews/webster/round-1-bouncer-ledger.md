---
round: 1
ledger:
  - key: join-leaves-timed-out-half-live
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: recipe-comment-redundant-clause
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: burler-help-overlong-line
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised three findings.
F1 (design, MEDIUM): `join` stops only the half that did not decide the round, so a half that ended `OutcomeTimeout` stays live and a retry can start a second session over it.
F2 (consistency, NIT): the three reworded `BurlerRound` comments in `contracts/recipes/loom-recipe.yaml` repeat their own opening.
F3 (consistency, NIT): one over-long line in the burler Long help in `internal/burlercli/cli.go`.
All three are open until a fresh review has seen their fixes.
