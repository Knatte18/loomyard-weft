---
round: 1
ledger:
  - key: card3-misses-claim-whys-on-carry-over
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card1-cites-untracked-decision-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: shared-decision-orchestrator-states-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 was the first round and had no prior ledger.
It raised three LOW findings, all non-gating:
F1 (Card 3 leaves two claim whys in `claims_test.go` describing a carry-over entry as an unreviewed fix),
F2 (Card 1 cites an untracked decision record as claim provenance),
and F3 (the one-rule-many-sites Shared Decision has the orchestrator state a rule Card 1 forbids it to restate).
The loop converged in round 1.
