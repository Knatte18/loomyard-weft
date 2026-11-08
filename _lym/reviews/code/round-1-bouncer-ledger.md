---
round: 1
ledger:
  - key: lym-spawns-bypass-lymbinary
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: button-reentry-raw-executable-and-doc-overclaim
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: process-cap-failure-path-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: refused-reexec-leaks-run-directory
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: missing-refusal-rows-sibling-reed-lines
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all new.
F1 asks either to convert the remaining `os.Executable` lym spawns or to amend the rule with a named exemption, so its fix may change a contract.
F2 is a local one-call fix plus a doc correction.
F3 to F5 are test and cleanup gaps with the fix text already fixed.
