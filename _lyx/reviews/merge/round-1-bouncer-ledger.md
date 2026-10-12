---
round: 1
ledger:
  - key: pauserecord-update-bypasses-state
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
---

Round 1 raised one finding, F1: the parent's `pauserecord.Update` rewrites `record.json` with its own marshal and atomic write, so unknown keys are lost.
The fix routes the rewrite through `state.UpdateJSON` and widens the leaf import set by `internal/state`.
