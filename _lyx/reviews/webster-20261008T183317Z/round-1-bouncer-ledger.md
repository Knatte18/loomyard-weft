---
round: 1
ledger:
  - key: send-retry-loops-count-uncapped
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: webster-reclaim-warn-dropped
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: send-config-comments-drift
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: send-never-typed-message
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: wait-header-stale-abandonstartup
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised five findings (F1 to F5), mapped in order to the keys above.
F1 (count-uncapped Enter, settle-read and unsent re-prompt loops) and F2 (webster's reclaim sites lost their Warn) gate.
The other three are LOW or NIT and do not gate.
