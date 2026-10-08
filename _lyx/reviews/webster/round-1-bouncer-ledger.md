---
round: 1
ledger:
  - key: stale-planoverviewframehash-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: readmerriamstart-uncovered-branches
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised two LOW consistency findings.
F1 maps to `stale-planoverviewframehash-comment`: comments in `state.go` and `doc.go` left stale by card 5.
F2 maps to `readmerriamstart-uncovered-branches`: a list-content launch line and a transcript ending at the prompt Read's result are untested.
No earlier ledger exists.
