---
round: 1
ledger:
  - key: passing-verify-row-skips-status-guard
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1: the "a passing verify runs no rerun" row has an empty log, so the Log guard in `rerunTestFailure` stops it before the Status guard.
The fix is local and fully specified in the finding (add `log: goFailure` to that row), so it is mechanical.
No prior ledger exists.
