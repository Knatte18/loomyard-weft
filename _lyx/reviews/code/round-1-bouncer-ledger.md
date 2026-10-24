---
round: 1
ledger:
  - key: terminal-recovery-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: absent-record-leaves-episode-open
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: recovery-message-misdescribes-live-again
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three low-severity findings on the waiterless-run logging.
F1 (`terminal-recovery-untested`) is a missing test stretch for the terminal-outcome recovery.
F2 (`absent-record-leaves-episode-open`) is a missing `Recover` call in the `!found` branch.
F3 (`recovery-message-misdescribes-live-again`) is an inaccurate recovery message.
None gates, so the loop converges.
