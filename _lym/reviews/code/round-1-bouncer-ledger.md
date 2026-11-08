---
round: 1
ledger:
  - key: unwire-sibling-test-lost-exclude-line-assertion
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1: the sibling-undirtied git test in `internal/fabricengine/unwire_test.go` lost its assertion on the real `unseedGitExclude` wiring.
The fix is a local assertion that the unwired pair's anchored exclude line is gone after Unwire.
No other finding was raised, and no earlier ledger exists.
