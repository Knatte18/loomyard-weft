---
round: 1
ledger:
  - key: d3-server-starting-boot-undefined
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d5-destroy-unattached-kills-session-before-pin
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d4-parse-error-applies-nothing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-open-ended-pin-clause
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: foreign-sessions-keep-server-alive
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-glob-metacharacters-in-path
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: tmux-conf-absence-bullet-misfiled
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised seven findings (F1 to F7, in the order of the keys above).
Four are design-class and gating: the undefined server-starting boot, the `destroy-unattached on` ordering failure, D4's parse-error premise and foreign sessions at teardown.
The rest are an unbounded pin clause, a glob rule that belongs in D2 and a misfiled exploration bullet.
All are open; there was no earlier ledger.
