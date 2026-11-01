---
round: 1
ledger:
  - key: watchfilechanges-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: floor-rest-after-change-partial-clock-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two test-coverage findings.
The first covers `watchFileChanges` and the watched path in `pullRequestDeps`, which have no test.
The second covers the floor rest after a change, pinned only with a standing clock.
Both fixes are local test additions whose content the findings spell out.
