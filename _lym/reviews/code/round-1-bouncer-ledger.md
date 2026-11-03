---
round: 1
ledger:
  - key: zero-exit-errwaitdelay-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: build-bounded-end-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: summary-overstates-start-time-check
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.
F1 and F2 are test-coverage gaps in `internal/gatecli`, one for the zero-exit `ErrWaitDelay` mapping and one for the build step's bounded end; each fix is fixed by the finding text.
F3 is a wording correction in the change description.
