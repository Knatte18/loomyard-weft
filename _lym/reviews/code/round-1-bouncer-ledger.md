---
round: 1
ledger:
  - key: card-fixer-notifier-never-fires
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: newshed-notify-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: recall-after-escalation-mails-again-undocumented
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: rownotice-wrapper-contracts-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rownoticerecord-missing-from-path-table
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: notify-parent-on-decoding-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: rows-only-narrowing-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: prefix-check-reads-first-notice-only
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, with no prior ledger.
Two MEDIUM design findings gate: the card fixer's hold notifier is dead code, and newShed's notify wiring has no test.
The rest are LOW test-pin and doc gaps, one NIT, and one `decision` finding deferred to the parent.
