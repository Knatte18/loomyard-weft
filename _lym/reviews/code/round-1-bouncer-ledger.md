---
round: 1
ledger:
  - key: stall-timer-default-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: silent-row-ready-eagain-retry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: answering-row-check-unproven
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unarmed-row-redundant-armed-assertion
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: serial-state-comment-omits-seam
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings in `internal/gitexec`'s test changes.
The gating one is the untested production `startStallTimer` default (F1).
The others are a readiness count that breaks when an EAGAIN start retry advances the attempt number (F2), an answering row that never proves its check ran (F3), a redundant assertion (F4) and an incomplete serial-state comment (F5).
Each fix text states its result, so all five are mechanical.
