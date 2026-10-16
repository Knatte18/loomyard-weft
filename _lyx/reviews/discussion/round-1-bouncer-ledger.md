---
round: 1
ledger:
  - key: d3-move-order-vs-reconcile
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-shell-window-last-pane
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-move-standing-or-one-shot
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: shuttle-acceptance-row-narrower-than-d2
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: grep-row-missing-anchorbelowparent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open.
The two gating ones concern D3's move: its order against reconcile's dead-pane kill and the Selvage heal, and its bound when the leftover strand pane is the shell window's only pane.
The rest are a standing-versus-one-shot ambiguity in D3 and two acceptance-row gaps.
