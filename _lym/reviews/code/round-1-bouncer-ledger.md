---
round: 1
ledger:
  - key: hubforge-bare-row-absent-identity
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised one NIT: the hubforge test's Bare row does not assert that `user.name` and `user.email` stay unset.
The fix is local and fully described by the finding text.
