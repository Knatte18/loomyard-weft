---
round: 1
ledger:
  - key: calibration-webster-engine-vs-master-engine
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1.
Calibration derives `RunTally.WebsterEngine` from the first located webster-role session, while the parent's live base uses the engine Master's run record names.
The review's fix text names the local correction, or a comment stating the single-engine assumption, so the fix is mechanical.
The finding is LOW and non-gating.
