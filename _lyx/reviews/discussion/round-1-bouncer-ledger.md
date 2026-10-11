---
round: 1
ledger:
  - key: d4-nil-cause-trigger-overbroad
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-background-omits-lone-percent-letthrough
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d5-misreads-daemon-wakeups-scope
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-dismissal-gap-edges-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d1-operator-hold-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d7-cap-governs-two-cases
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

First round; all six findings are new.
D4's pane-tail trigger conflates a nil cause with never-ready.
D5 rests on a premise the daemon-wakeups background does not grant.
D2 and D1 state the lone-`%` let-through and the operator-hold scope inconsistently with the invariant wording.
D6 and D7 have unstated edges and an over-narrow premise.
