---
round: 1
ledger:
  - key: d7-row-time-measured-from-wrong-end
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: no-strand-absorbs-free-rows-after-orch-stop
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: notice-log-trigger-misses-held-back-and-dropped
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: anchor-kind-combinations-without-height
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d2-d7-unstated-state-outcomes
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: lyx-cleanup-risk-is-a-constraint
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: deliberation-narrative-in-decision-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: missing-acceptance-criteria
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised eight findings, all open.
The two BLOCKING design findings are the row-time rule in D7 and the unabsorbed free rows after `lyx orch stop` in D1 and D3.
The two MEDIUM decision findings are the notice-log trigger in D8 and the undefined anchor and kind height combinations in D3.
The four LOW findings are unstated state outcomes in D2 and D7, the cleanup risk that is a constraint, deliberation narrative to relocate, and missing acceptance criteria.
