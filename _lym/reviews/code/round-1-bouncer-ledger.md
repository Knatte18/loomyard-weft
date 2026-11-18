---
round: 1
ledger:
  - key: retype-watch-offset-includes-older-operator-turn
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: retype-latched-before-send-busy-uses-it-up
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: no-input-box-reading-not-checked
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: fresh-start-retype-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: busy-retry-bounds-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: new-sends-not-cut-to-run-deadline
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: retype-guards-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failure-capture-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: abandonstartup-doc-wrong-capture-source
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: promptdelivered-comment-overstates-latch
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised ten findings, all open.
The gating ones are the retype watch offset, the retype latch set before the send, and the missing tests for the fresh-start retype and the busy retry bounds.
The no-input-box reading check is a scope MEDIUM, and the rest are LOW or NIT.
