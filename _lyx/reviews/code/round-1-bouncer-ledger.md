---
round: 1
ledger:
  - key: compare-marker-no-retry
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: ruling-comments-wrong-scope
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: cancel-skips-capture-kill
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: capture-head-read-before-lock
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all open.
F1 (`compare-marker-no-retry`) is the gating one: `RunCompare` needs the bounded-retry take that `RunCapture` uses.
F2 to F4 are local corrections: the ruling comment scope wording, a `killCapture` call in the cancel path, and reading HEAD under the capture lock.
