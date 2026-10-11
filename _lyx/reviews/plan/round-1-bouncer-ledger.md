---
round: 1
ledger:
  - key: parseseed-test-economy-repeat
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: shed-span-timestamp-and-absent-seed-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: all-runs-sum-and-paragraph-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: shedrun-doc-claim-vs-treerunfilerels
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised four findings, all open.
F1 (`parseseed-test-economy-repeat`) is the gating one: four of `TestParseSeed`'s six rows repeat existing `ReadSeed` tests.
F2 is the missing test inputs for card 4's timestamp-parse abort and absent-seed branch.
F3 is the unpinned "All runs" sums in card 2.
F4 is the shedrun package doc and `paths.go` header contradicted by `TreeRunFileRels`.
