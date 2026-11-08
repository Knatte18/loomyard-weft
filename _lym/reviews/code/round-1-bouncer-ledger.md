---
round: 1
ledger:
  - key: recipe-ban-unqualified-actor
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one LOW consistency finding (F1): the board-policy bullet's recipe-change ban names no actor,
so it collides with the rule that the operator may name darn for a task.
The fix is a local rewording that binds the ban to the session.
