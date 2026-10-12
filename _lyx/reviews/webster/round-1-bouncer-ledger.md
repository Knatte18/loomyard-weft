---
round: 1
ledger:
  - key: own-window-relaunch-leaves-dead-pane
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: placement-constants-mixed-iota
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised two findings.
F1 maps to `own-window-relaunch-leaves-dead-pane`: a relaunch rebinds the strand to a new pane and never kills the old one, which stays in a shared group window.
F2 maps to `placement-constants-mixed-iota`: the placement constants mix `iota` with literal values.
