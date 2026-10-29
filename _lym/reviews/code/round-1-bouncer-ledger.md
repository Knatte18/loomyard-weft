---
round: 1
ledger:
  - key: spawn-test-serial-tests-unnamed-global-state
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding, F1: two serial launch tests in `internal/reedengine/spawn_test.go` capture the process-global logger without a comment naming it.
The fix is a one-line doc comment per test, fixed by the finding text.
No earlier ledger exists.
