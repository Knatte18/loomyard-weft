---
round: 1
ledger:
  - key: smoke-test-vacuous-when-denied-command-not-tried
    rounds: [1]
    status: open
---

Round 1 raised one LOW consistency finding (F1): the smoke test in `internal/shuttlecli/smoke_bashstdin_test.go` passes when the agent never attempts the denied reset.
It stays open because no fresh reviewer has seen the fix.
