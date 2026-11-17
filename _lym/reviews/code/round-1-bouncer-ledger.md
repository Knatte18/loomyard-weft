---
round: 1
ledger:
  - key: writehold-former-name-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: upgrade-notice-repeats-on-rerun
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: bolthold-entry-keeps-first-machine
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: owner-caller-described-as-live
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: writehold-second-hold-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised five findings (F1 to F5), none seen before.
F1 maps to `writehold-former-name-untested`, F2 to `upgrade-notice-repeats-on-rerun`, F3 to `bolthold-entry-keeps-first-machine`,
F4 to `owner-caller-described-as-live` and F5 to `writehold-second-hold-unpinned`.
Only F1 gates.
F5 is marked `design-wrong` because its fix depends on a contract the finding leaves open.
