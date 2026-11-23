---
round: 1
ledger:
  - key: webster-state-filename-literal
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: run-record-check-holds-by-construction
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised two findings against `internal/loomcli/amendduringwebster_test.go`.
F1 (`webster-state-filename-literal`): the test spells `state.json` instead of reading the record through `websterengine.LoadState`.
F2 (`run-record-check-holds-by-construction`): neither the verb's seams nor the bare goto reach the planted record, so the byte check cannot fail.
Both carry exact fix text in the review.
