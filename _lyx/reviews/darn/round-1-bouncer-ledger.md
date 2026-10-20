---
round: 1
ledger:
  - key: store-race-error-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: chtimes-on-pruned-entry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: non-hub-tagged-path-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: trace-id-literal-in-shedverbs
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings, all first seen this round.
The MEDIUM design finding on untested store race and error paths gates the round.
The other three are LOW or NIT and each has a local fix the finding text fixes.
