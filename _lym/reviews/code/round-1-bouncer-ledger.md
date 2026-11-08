---
round: 1
ledger:
  - key: terminal-read-refusal-names-unprinted-prompt
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: read-hidden-terminal-line-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 (the terminal-read error's way forward mentions a prompt that may not have printed) and F2 (the no-terminal result of `readHiddenTerminalLine` has no test).
Both are LOW and local; the first round has no earlier ledger to carry forward.
