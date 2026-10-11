---
round: 1
ledger:
  - key: driver-busy-notice-state-text-uncovered
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: pairrelative-strips-prefix-inside-longer-path
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 raised two LOW findings.
F1: the driver-busy notice's leading state text has no assertion in the busy-notice test.
F2: `pairRelative` strips the pair prefix at any position, not only where a path starts.
Neither gates.
