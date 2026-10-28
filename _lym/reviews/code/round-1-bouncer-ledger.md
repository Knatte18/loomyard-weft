---
round: 1
ledger:
  - key: waiterless-run-counted-live
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: carryover-selection-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: path-prompt-read-as-slash-command
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: first-instruction-dropped-when-prompt-undelivered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: carryover-failure-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all `design`: two MEDIUM (F1, F2) and three LOW (F3, F4, F5).
F1 widens the reading of waiterless running records beyond the idle notice to the loop-arm activity check and decision notice recipients.
F2 is the carry-over test table missing died and timeout predecessors and the newest-run choice.
F3 to F5 are a path-opening prompt read as a slash command, a dropped first instruction when the prompt never landed, and untested failure and retry paths.
