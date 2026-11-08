---
round: 1
ledger:
  - key: draft-overwrites-existing-description
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: describe-unreadable-landing-record-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: describe-draft-flag-wiring-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.
F1 (`draft-overwrites-existing-description`) is the gating one: `describe --draft` writes `summary.md` unconditionally, so it can destroy an orch-edited or recorded description.
Its fix text names the refusal, the test row and the doc updates, so it is mechanical.
F2 and F3 are test-coverage gaps with fixes the findings spell out.
