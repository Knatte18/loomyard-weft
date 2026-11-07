---
round: 1
ledger:
  - key: widest-stamped-line-omits-repository
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: sending-repository-replay-carry-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: sending-repository-envelope-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: reinterpretation-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: transient-resolve-failure-drops-trust-entry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: repository-segments-unchecked-before-stamp
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all class `design`: four MEDIUM (F1 to F4) and two LOW (F5, F6).
F1 to F4 map to the first four keys.
F5 maps to the transient-failure key, and its fix changes which resolve errors drop an entry, so it needs a fresh review.
F6 maps to the last key.
No earlier ledger exists.
