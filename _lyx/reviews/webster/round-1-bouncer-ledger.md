---
round: 1
ledger:
  - key: runtargeting-cites-deleted-composeprompt
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: pathrules-comment-names-deleted-test
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW scope findings, both stale comment references to code the diff deleted: F1 (`runTargeting` doc comment) and F2 (`pathRules` comment).
Both are local one-line corrections whose text the findings fix.
No gating finding; no earlier ledger.
