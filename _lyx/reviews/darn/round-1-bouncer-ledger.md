---
round: 1
ledger:
  - key: claudeengine-doc-stale-quote-aware-rows
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: double-quoted-substitution-passes-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: lt-forms-and-heredoc-unreached-commands
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: new-row-test-gaps
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1, first ledger.
F1 maps to `claudeengine-doc-stale-quote-aware-rows`, F2 to `double-quoted-substitution-passes-rows`, F3 to `lt-forms-and-heredoc-unreached-commands`, F4 to `new-row-test-gaps`.
Each fix is a local correction the finding text already specifies.
