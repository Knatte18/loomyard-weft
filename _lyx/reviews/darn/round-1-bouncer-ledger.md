---
round: 1
ledger:
  - key: operand-sentence-pin-dropped
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding.
The coverage lens still carries the operand sentence, but the pinned-sentence check in `internal/lenses/config_test.go` no longer pins it.
The review's fix text names the exact change, so the fix is mechanical.
