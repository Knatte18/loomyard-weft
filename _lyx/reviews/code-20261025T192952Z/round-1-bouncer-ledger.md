---
round: 1
ledger:
  - key: dobby-run-cells-stale-pair-location
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: runs-window-title-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 reviewed the Parent-Align amendment and raised two LOW findings.
F1 maps to `dobby-run-cells-stale-pair-location`: the cached pair Location outlives an anchor move.
F2 maps to `runs-window-title-wiring-untested`: the `runsWindowTitleOf` wiring has no test.
Neither is gating, so the loop converges.
