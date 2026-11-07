---
round: 1
ledger:
  - key: packed-template-test-misses-pack-refs
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cruft-repack-unrecorded-departure
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW findings.
The first is a test gap: nothing pins `pack-refs --all` or the cruft pack.
The second is the `--cruft` departure from the record's `git repack -ad`, which only the run's parent can record.
Neither gates.
