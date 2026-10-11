---
round: 1
ledger:
  - key: marker-write-vs-host-write-scope
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-marker-list-misses-host-specific-refs
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d6-bound-covers-record-only-and-nested-forks
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: relaunched-writer-lost-forks-undefined
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: poll-misreads-daemon-wakeups-and-fswatch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-carries-provenance-and-consumer-list
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: missing-criteria-d3-reusability-d2-write-scope
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised seven findings, none BLOCKING.
Four are `design` at MEDIUM and gate: the write-scope conflict with the host template, the D6 bound and nested forks, the relaunched writer, and the poll's PATTERN basis.
The rest are a `consistency` MEDIUM on D3's marker list, a `consistency` LOW on D9's provenance and caller list, and a `scope` LOW on two missing criteria.
