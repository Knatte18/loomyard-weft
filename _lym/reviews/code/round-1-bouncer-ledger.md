---
round: 1
ledger:
  - key: stat-only-locator-overclaimed-docs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reached-parent-wiring-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW findings.
F1 (`stat-only-locator-overclaimed-docs`): `presentTaskWorktreePath` still tests only `os.Stat`, so the docs claiming one answer for every verb overclaim.
F2 (`reached-parent-wiring-untested`): no test drives `hubStartGuard`'s real `ReachedParent` wiring through the reached case.
Both fixes are local and spelled out in the finding text.
