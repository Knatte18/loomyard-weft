---
round: 1
ledger:
  - key: headcontains-slack-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: pattern-hub-store-gc-understates-writers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised two LOW non-gating findings.
F1 maps to `headcontains-slack-untested`: no test pins the 24-hour slack of the `HeadContains` bound.
F2 maps to `pattern-hub-store-gc-understates-writers`: the `PATTERN-hub-store-gc` overview line omits that `Topology.Add` writes the keys too.
