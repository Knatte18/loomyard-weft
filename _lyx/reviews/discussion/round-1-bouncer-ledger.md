---
round: 1
ledger:
  - key: d1-timeout-unstartable-no-failure-record
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d3-crashed-driver-leftover-still-refused
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-standalone-run-slug-free-premise
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-hardcoded-hub-shortname
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-conditional-fail-branch-undesigned
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d1-bound-overstates-reverify
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d2-bound-misses-hand-started-merge-in
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: acceptance-pins-already-built-behavior
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: constraints-carry-no-requirement
    rounds: [1]
    status: open
    class: scope
    severity: NIT
  - key: open-risks-restate-bounds
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

First round: no prior ledger.
All ten findings of the round-1 review are carried as open.
The gating ones are the D1 route for timed-out and unstartable verifies (no failure record is written for them), the D3 guard that still refuses a crashed-driver leftover, and D6's two false premises (standalone runs, hard-coded shortname).
