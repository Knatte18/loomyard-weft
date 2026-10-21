---
round: 1
ledger:
  - key: merged-overview-drops-parent-wording
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: declared-origin-reconcile-text-shortened
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: linked-scratch-untested-under-local-removal
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, none `design`.

- F1: the merged `PATTERN.md` overview dropped the parent's wording of `PATTERN-hub-wide-config`, `PATTERN-hub-store-gc` and `PATTERN-friction-capture`.
- F2: the merge shortened the declared `PATTERN-origin-reconcile` text.
- F3: no test pins a populated, linked root `.scratch` under local removal, snapshot and the dirty gate.

Each fix is a local correction whose result the finding text fixes, so all are marked `mechanical`.
