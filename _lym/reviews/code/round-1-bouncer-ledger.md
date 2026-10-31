---
round: 1
ledger:
  - key: run-turn-starts-raw-skip-undocumented
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hold-notice-operator-skip-unpinned-on-deferral
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: doc-go-carry-over-reference-unresolved
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

First round.
F1 maps to `run-turn-starts-raw-skip-undocumented`, F2 to `hold-notice-operator-skip-unpinned-on-deferral`, F3 to `doc-go-carry-over-reference-unresolved`.
All three are LOW with a fix text that fixes the result, so none is gating.
