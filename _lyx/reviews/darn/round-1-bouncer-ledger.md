---
round: 1
ledger:
  - key: stencil-wrong-profiles-command
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: light-darn-block-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: stencil-semicolon-line-break
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 (F1, F2, F3) is the first ledger.
F1: the stencil names `lyx config profiles` where `lyx board profiles` lists the hub's profiles.
F2: no assertion pins the `light` profile's darn block.
F3: a semicolon clause sits on one line in the new stencil bullet.
All three are local corrections whose text the findings fix.
