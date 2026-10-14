---
round: 2
ledger:
  - key: deleting-cards-reported-as-later-holders
    rounds: [1]
    status: resolved
---

Round 1 raised one finding (F1): deleting cards joined the Edit-region set that also feeds the reported holders.
Round 2 found it fixed: `LaterDeleteReferences` keeps deleting cards only as exemptions, and a test row pins the case.
Round 2 raised no new findings.
