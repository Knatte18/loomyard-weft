---
round: 1
ledger:
  - key: rejected-model-strand-left-live
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stale-api-error-needs-retries-docs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: handback-text-claims-resends-spent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: rejected-branch-missing-done-and-pause-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: run-envelope-apierror-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, F1 to F5, in the order of the ledger.
F1 is the only gating one: the at-once api-error end for a rejected model returns promptly, but `finalize` removes the strand only for done, so the strand stays live.
F2 and F3 are wording drift in docs and hand-back text.
F4 and F5 are coverage gaps in the shuttleengine and shuttlecli tests.
Each fix is spelled out in the review's finding text, so all are mechanical.
