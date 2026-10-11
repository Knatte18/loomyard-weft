---
round: 1
ledger:
  - key: sandbox-caller-list-misses-f11-mislabels-f15
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: fabricengine-doc-go-unwire-stale
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: unwire-integration-test-duplicates-engine-coverage
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: floor-value-shapes-pinned-twice
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: migrateconfig-engine-doc-go-not-updated
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised five findings, none BLOCKING.
Two are `design` (test economy in cards 5 and 6) and gate the loop; three are `scope` gaps (sandbox caller list in card 6, `doc.go` updates in cards 5 and 6).
