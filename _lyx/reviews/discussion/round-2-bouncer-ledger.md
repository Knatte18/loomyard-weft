---
round: 2
ledger:
  - key: d3-move-order-vs-reconcile
    rounds: [1]
    status: resolved
  - key: d3-shell-window-last-pane
    rounds: [1]
    status: resolved
  - key: d3-move-standing-or-one-shot
    rounds: [1]
    status: resolved
  - key: shuttle-acceptance-row-narrower-than-d2
    rounds: [1]
    status: resolved
  - key: grep-row-missing-anchorbelowparent
    rounds: [1]
    status: resolved
  - key: d3-shell-window-identity-without-selvage
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-reconcile-relist-after-move
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: grep-row-missing-retired-top
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's five findings no longer apply: round 2's review settles the move's order against reconcile and the Selvage heal, the only-pane Bound, and the standing-step question, and raises nothing on the shuttle acceptance row or the `AnchorBelowParent` grep entry.
Round 2 raised three new keys.
The gating one is that D3's read-time remap leaves the shell window unidentified when Selvage is missing.
The others are the missing re-listing after the move and the grep row's omission of the retired `top` value.
