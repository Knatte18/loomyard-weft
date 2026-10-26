---
round: 1
ledger:
  - key: auxv-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1: no test pins that `paneProcessAge` reads the real auxv tick rate.
It is LOW, below the gating threshold, and its fix text fully specifies the added test.
