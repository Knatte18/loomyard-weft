---
round: 1
ledger:
  - key: boot-window-log-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: doc-claims-every-window-named
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: window-name-log-field-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all local corrections whose text fixes the result.
The boot-window log lacks a test for its Info line and non-fatal read failures (F1, MEDIUM, gating).
The package doc overstates that every window is opened with a name (F2, LOW).
The new `name` log field on the own-window and break-out lines is unpinned (F3, LOW).
