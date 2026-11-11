---
round: 1
ledger:
  - key: hubland-heldslugs-wiring-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding, F1: the `hubLandDeps` `HeldSlugs` closure, which unions run directories and task branch slugs, has no test that reaches it.
The fix is a local test addition whose content the finding text fixes, so it is mechanical.
It is non-gating, and no earlier ledger exists.
