---
round: 1
ledger:
  - key: gate-time-utc-conversion-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised one LOW `design` finding, F1: no `TestRenderChecklist` row feeds a gate span with a non-UTC offset, so `.UTC()` in the gate time formatting is unpinned.
The fix is local and fully specified by the finding text.
No earlier ledger exists, so nothing carries forward.
