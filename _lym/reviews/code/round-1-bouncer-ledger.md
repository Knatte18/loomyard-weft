---
round: 1
ledger:
  - key: refuse-prefix-longer-than-compile-form-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overview-states-compile-check-steer-unconditionally
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised two findings, neither gating.
F1 is a coverage gap: `refusedRaw`'s length guard has no test row for a refuse prefix longer than the compile form.
F2 is a doc wording gap: the overview states the compile-check steer without the `refuse_raw` condition.
Both fixes are local and spelled out in the review.
