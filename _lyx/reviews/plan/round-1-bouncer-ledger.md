---
round: 1
ledger:
  - key: card-18-socket-sources-surveyed
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: killoncleanup-unbounded-removal
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: overview-no-test-file-before-card-9
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card-20-no-reuse-of-card-1-aggregation
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card-10-tiersleep-retired-id
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first round, so there is no earlier ledger.
Five findings are open; none is gating.
The scope MEDIUM asks card 18 to name the known raw `kill-server` test files; the rest are LOW or NIT wording and bound fixes.
