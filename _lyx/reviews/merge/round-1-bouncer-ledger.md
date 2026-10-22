---
round: 1
ledger:
  - key: loom-webster-gate-unheld
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: webster-gate-hold-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised two findings, both in the task's own seam with the merged `wire`.
`loom-webster-gate-unheld` (F1): `wire` in `internal/loomcli/wiring.go` leaves `Geometry.VerifyHold` nil, so the loom run's webster gate runs unheld under a pause.
`webster-gate-hold-untested` (F2): nothing in `internal/websterengine` tests that the gate tells the hold on the first run and the rerun.
Both fixes are local corrections the finding text already fixes.
