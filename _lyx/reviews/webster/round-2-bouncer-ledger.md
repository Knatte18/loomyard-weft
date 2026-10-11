---
round: 2
ledger:
  - key: reed-stale-socket-test-fork-race
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
---

Round 1 raised nothing.
Round 2 raised one BLOCKING finding: the stale-socket rows in `internal/reedengine/socketfile_test.go` close a non-unlinking listener, and a concurrent fork can keep the closed socket answering `Dial`, so `removeStaleSocket` correctly keeps the file and the row fails under a loaded parallel run.
Key `reed-stale-socket-test-fork-race` stays open until a fresh review sees the fixture change.
