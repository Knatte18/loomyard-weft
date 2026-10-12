---
round: 2
ledger:
  - key: smoke-driverstrand-stale-status-strand-assertion
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
  - key: smoke-starttail-stale-status-strand-assertion
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
---

Round 1 raised nothing.
Round 2 raised two BLOCKING `scope` findings from the Publish verify failure: two `internal/loomcli` smoke tests still assert zero status strands after an llm-seeded start, while the branch's card 7 makes start ensure the `loom-status` strand on both drivers.
The merge-in did not cause either failure.
Both stay open until a fresh review confirms the assertions.
