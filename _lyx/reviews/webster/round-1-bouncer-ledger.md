---
round: 1
ledger:
  - key: stale-removeportal-test-comments
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: undeclared-pattern-rewording
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 (stale `removePortal` and "portal/launcher" wording in two test comments) and F2 (three PATTERN entries reworded without a declaration).
Both are LOW and mechanical, and neither is gating.
