---
round: 1
ledger:
  - key: summary-misdescribes-empty-pack-prior-behavior
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding (F1): `_lyx/landing/summary.md` says a batch with no entries previously got a non-nil empty pack, but it rendered the pack-on prompt.
The fix is a local wording correction in the summary.
No gating finding is open.
