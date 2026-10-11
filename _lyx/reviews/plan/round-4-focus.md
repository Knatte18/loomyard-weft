---
round: 4
focus:
  - "Card 5 (`_lyx/plan/05-convergence-guard.md`): enumerate every test fixture in `internal/shedadapters`, `internal/shedrecipe` and `internal/loomrecipe` that reaches the guard's ledger key check on any path, harvest and replay (a CONTINUE or CIRCLING round on disk at Call entry), for every verdict word. Settle: does each fixture whose round-2-or-later ledger drops a previous round's key appear under `**Edit:**` and in `**ImpactSummary:**`?"
  - "Card 5's Intent claims about which ledgers keep their previous round's keys: verify each claim against every fixture that writes a round-2-or-later ledger, including `TestBouncer_CirclingGuard` rows. Settle: is any claim false for a fixture the card does not target?"
  - "For each fixture row the card changes, does the Intent state the new row content so the row keeps its intended outcome (circling earned or unearned) under the guard?"
---

The method gap is enumerating by verdict word or by grepping for one severity.
Walk each settle path the guard runs on and read the fixture ledgers it sees.
