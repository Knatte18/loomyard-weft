---
round: 1
ledger:
  - key: paused-intervals-grace-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: summary-overstates-grace-trigger
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: battenshed-doc-driver-dead-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: dobbyengine-takeover-bound-omits-grace
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings, no prior ledger.
F1 (design, MEDIUM) is the one gating finding: a test row for paused intervals in the start grace.
F2 to F4 are doc and summary wording corrections.
