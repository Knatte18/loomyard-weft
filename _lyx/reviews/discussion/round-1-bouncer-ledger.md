---
round: 1
ledger:
  - key: d1-interpreter-wording-fails-no-denied-recovery-scan
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: d6-latch-never-clears-after-first-notice
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-restart-rebuild-repeats-sent-notice
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d2-sed-word-extent-and-incomplete-bound
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d7-closed-list-covers-only-bare-in-place-spellings
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d3-bound-omits-positional-parameters
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: goal-claims-more-steered-forms-than-d3-covers
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d1-exclude-entry-has-no-stated-reach
    rounds: [1]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised eight findings, all open, with no earlier ledger.
The gating ones are the BLOCKING D1 wording that breaks the `PATTERN-no-denied-recovery` scan and the design-class D6 latch that never clears.
The D6 restart rebuild, D2 sed-word extent, D7 closed-list gap, D3 positional parameters, Goal over-claim and D1 exclude reach are the remaining findings.
