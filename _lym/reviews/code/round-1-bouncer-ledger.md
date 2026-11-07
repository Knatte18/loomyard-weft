---
round: 1
ledger:
  - key: summary-newwiredpairfixture-site-count
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: summary-beamhub-site-count
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW consistency findings, both about site counts in the change description that the code does not bear out.
F1 maps to `summary-newwiredpairfixture-site-count` and F2 to `summary-beamhub-site-count`.
No earlier ledger exists, and no finding is gating.
