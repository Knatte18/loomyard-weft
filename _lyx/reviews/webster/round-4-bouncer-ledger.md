---
round: 4
ledger:
  - key: parserless-engine-slash-waits-submit-window
    rounds: [1]
    status: resolved
  - key: stale-session-start-line-untested
    rounds: [1]
    status: resolved
  - key: pending-call-window-widening-untested
    rounds: [1]
    status: resolved
  - key: integration-driver-bootstrap-stub-no-session-start
    rounds: [3]
    status: resolved
  - key: smoke-driver-strand-stub-no-session-start
    rounds: [3]
    status: resolved
  - key: smoke-start-tail-shares-driver-stub
    rounds: [3]
    status: resolved
---

Rounds 1 and 2 closed the three round-1 findings.
Round 3 found the stub providers in `internal/loomcli` tests never recorded a `SessionStart` line.
Round 4 reviewed the fixes: both stubs now run `stubRecordSessionStart`, the three failing tests pass, and no finding remains.
