---
round: 1
ledger:
  - key: retired-deny-agent-tool-key-move
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: headless-claude-bundled-short-flags
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: duplicate-agent-absence-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; there is no earlier ledger.
F1 maps to `retired-deny-agent-tool-key-move`, F2 to `headless-claude-bundled-short-flags`, F3 to `duplicate-agent-absence-assertion`.
All three are LOW or NIT, so the loop converges with them carried as open.
