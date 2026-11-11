---
round: 1
ledger:
  - key: doc-go-refusal-line-unqualified
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 only: `internal/boardcli/doc.go` line 114 states the old refusal rule without marking the beam-orch case as the exception.
The fix is a local rewording the finding text already fixes.
