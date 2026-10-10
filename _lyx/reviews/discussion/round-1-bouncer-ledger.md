---
round: 1
ledger:
  - key: advisor-start-failure-notice-before-chair-exists
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: send-notice-needs-idle-chair
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: done-advisor-strand-removed
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: advisor-to-parent-route-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: still-live-undefined
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: provider-word-single-source-claims-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: added-after-discussion-duplicates-d6
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: decisions-relitigate-rejected-alternatives
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eight findings, none carried from an earlier ledger.
The three BLOCKING `design` findings concern D7's and D5's advisor lifecycle and notice delivery: a notice before the chair exists, `Send` needing an idle chair, and shuttle removing a done advisor's strand.
The MEDIUM `design` finding is the advisor-to-parent route that D6 leaves unbounded.
The rest are `consistency` findings: the undefined "still live" (MEDIUM), false provider-word claims, the duplicated "Added after Discussion" section and re-litigated alternatives (LOW).
