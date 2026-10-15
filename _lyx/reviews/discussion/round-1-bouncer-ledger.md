---
round: 1
ledger:
  - key: d5-held-span-kind-double-count
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-carrier-set-misses-burler-review-gate
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-open-rework-visit-attribution
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: d4-child-number-undefined-for-review-segment-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-examples-contradict-d3-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: auto-mode-assumption-restates-d5
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, none of them in an earlier ledger.
Two are gating design findings in the gate-span decisions: D5 leaves the held-pending time's span kind open, so D6 can double count, and D5's carrier set starts from today's count copiers and misses the Burler review row's gate.
D7's open rework visit is a LOW decision gap.
The two D4 numbering findings and the redundant Auto-mode assumption are local corrections.
