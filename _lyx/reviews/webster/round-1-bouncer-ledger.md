---
round: 1
ledger:
  - key: testtiming-keep-directive-detached
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding: the `testtiming:keep` directive in `internal/orchengine/state_test.go` sits above the new `TestPeekState` instead of `TestCycleRequest_WriteProbeClear`.
The fix is a local move of one line, fixed by the finding text.
