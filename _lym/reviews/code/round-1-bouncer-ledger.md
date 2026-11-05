---
round: 1
ledger:
  - key: arbiter-notice-before-consume
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: arbiter-hard-cap-record-and-accept
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: decision-finding-parent-only-prose
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: arbiter-notice-paths-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: arbiter-decision-rule-refusal-check
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: arbiter-brief-override-unspecific
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 opened six findings (F1 to F6, in the order above).
The two gating ones concern the arbiter's outcome notice timing and the hard-cap accept rule.
The rest are prose consistency, test coverage of the notice, the refusal check, and the brief override list.
