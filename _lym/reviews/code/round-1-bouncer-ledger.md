---
round: 1
ledger:
  - key: anchor-dir-existence-unchecked
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prompt-failure-message-unpinned-to-moved-anchor
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: info-log-symlink-false-anchor-change
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; there was no prior ledger.
Three findings, all local corrections whose fix text is already specified.
The highest severity is LOW, and the one design-class finding is below the gating threshold.
