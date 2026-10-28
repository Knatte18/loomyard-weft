---
round: 1
ledger:
  - key: haltreason-doc-escalate-prefix
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round and has no prior ledger.
F1 maps to `haltreason-doc-escalate-prefix`: the `haltReason` doc in `internal/landingshed/fixerrun.go` opens with a sentence a loom or darn escalate contradicts.
The review gives the exact replacement sentence, so the fix is mechanical.
