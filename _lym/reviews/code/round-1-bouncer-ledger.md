---
round: 1
ledger:
  - key: sweep-edit-unbounded-failure-fix
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: added-assertions-on-static-stencil-prose
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: sweep-grep-overlaps-rename-grep
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 (sweep edit called a failure fix with no bound), F2 (two assertions on static stencil prose) and F3 (sweep grep overlapping the Rename paragraph's grep).
All are consistency findings with fixes the review text already fixes.
