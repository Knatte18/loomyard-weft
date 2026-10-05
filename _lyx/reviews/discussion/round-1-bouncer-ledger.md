---
round: 1
ledger:
  - key: d4-voucher-lifetime-and-write-paths
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d5-drop-rule-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d2-loader-question-already-settled
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: duplicated-removal-and-field-lists
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

First round; no prior ledger.
Four findings, all open, none design-class.
D4's voucher lacks a stated lifetime and write-path bound (F1), D5's agent-judged drop rule lacks a stated bound (F2), D2 and Q8 pose a loader question the loader already answers (F3), and Scope-In, D1, D3 and the acceptance criteria repeat the same lists (F4).
