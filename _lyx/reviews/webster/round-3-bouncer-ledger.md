---
round: 3
ledger:
  - key: reed-stale-socket-test-fork-race
    rounds: [2]
    status: resolved
---

Round 1 raised nothing.
Round 2 raised one BLOCKING finding: the stale-socket rows in `internal/reedengine/socketfile_test.go` closed a non-unlinking listener, and a concurrent fork could keep the closed socket answering `Dial`.
Commit `ea4e53819` moved those rows onto a datagram-bound socket, and round 3 found nothing, so `reed-stale-socket-test-fork-race` is resolved.
