---
round: 1
ledger:
  - key: p0-label-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: p0-row-ignores-done-entries
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: p0-row-place-under-sort
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stepprojects-doc-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all open.
F1 maps to `p0-label-branch-untested`, F2 to `p0-row-ignores-done-entries`, F3 to `p0-row-place-under-sort`, F4 to `stepprojects-doc-comment-stale`.
Each finding text fixes its own correction, so all four are mechanical.
