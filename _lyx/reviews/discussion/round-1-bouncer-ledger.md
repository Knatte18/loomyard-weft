---
round: 1
ledger:
  - key: d2-refusal-stale-verify-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-bound-no-retry-loss
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d4-target-file-match-and-bound
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d4-pr-narrative-wrong-stencil
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-landing-order-source
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d5-bound-semantic-overlap
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d1-pause-check-hand-cleared
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised seven findings, all open.
The gating one is D2's refusal reading a verify-failure record that outlives its halt.
The D4 file-level target match and the D5 row-order source are MEDIUM decision gaps.
The remaining four are LOW bound and wording gaps in D1, D2, D4 and D5.
