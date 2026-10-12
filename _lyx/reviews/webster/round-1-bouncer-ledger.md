---
round: 1
ledger:
  - key: loomcli-stale-loader-shape-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: local-file-never-disposable-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: runstatusproducer-test-duplicates-subtest
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first round, with no prior ledger.
It raised three LOW consistency findings: a stale comment in `internal/loomcli/wiring.go` (F1), an absolute local-file rule in the docs that the code's two removal paths contradict (F2), and a duplicate top-level test in `internal/boardengine/runlock_test.go` (F3).
None is gating.
