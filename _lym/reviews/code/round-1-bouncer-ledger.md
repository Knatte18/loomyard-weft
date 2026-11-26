---
round: 1
ledger:
  - key: overview-names-bare-relay-commands
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: own-post-user-id-clause-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: presence-word-with-extra-words-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised one stale doc sentence about the relay commands and two missing test rows, in own-post detection and in relay-command parsing.
Each finding's text fixes the result of its correction.
None gates.
