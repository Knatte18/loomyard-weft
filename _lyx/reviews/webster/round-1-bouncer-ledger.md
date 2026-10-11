---
round: 1
ledger:
  - key: empty-home-tilde-expands-to-root
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: refused-boot-leaves-no-server-unasserted
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: test-isolation-bullet-placement
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first round, so there is no prior ledger.
F1 maps to `empty-home-tilde-expands-to-root`, F2 to `refused-boot-leaves-no-server-unasserted`, F3 to `test-isolation-bullet-placement`.
All three are below the gating threshold.
