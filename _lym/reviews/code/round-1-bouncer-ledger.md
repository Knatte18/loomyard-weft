---
round: 1
ledger:
  - key: script-argument-scan-overrun
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: keywords-widen-top-level
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: claudeengine-doc-bash-c-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: raw-go-quote-word-end-top-level
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: script-misses-expansion-and-heredoc
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: new-behaviour-without-rows
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings.
F1 (`script-argument-scan-overrun`) gates: argument-word scans after a script-reached command word cross the script's closing quote.
F4 and F5 share its root, the script branch not being read like the top level, and are LOW.
F2 needs a parent decision, F3 is a doc correction, F6 is missing test rows.
