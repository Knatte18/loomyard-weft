---
round: 1
ledger:
  - key: batten-usage-limit-precedence-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: shuttle-activity-usagelimit-copy-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: reset-for-fresh-launch-limit-fields-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: tick-usage-limit-failure-paths-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: past-reset-waits-full-retry-interval
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: impossible-month-day-reads-as-date
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: wake-inject-ttl-zero
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger carries forward.
Seven findings, none gating: three MEDIUM scope test gaps (F1, F2, F3), one LOW scope test gap (F4), two LOW design findings (F5, F6) and one LOW consistency finding (F7).
