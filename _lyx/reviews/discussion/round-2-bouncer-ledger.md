---
round: 2
ledger:
  - key: machine-layer-anchor-without-lyx
    rounds: [1]
    status: resolved
  - key: d3-message-reaches-board-only-checks
    rounds: [1]
    status: resolved
  - key: retired-machine-key-two-outcomes
    rounds: [1]
    status: resolved
  - key: d3-from-prime-and-removal-hatch-bound
    rounds: [1]
    status: resolved
  - key: every-hub-wide-module-gains-machine-layer
    rounds: [1]
    status: resolved
  - key: editor-save-outcome-unstated
    rounds: [1]
    status: resolved
  - key: d2-bound-misstates-direct-write
    rounds: [1]
    status: resolved
  - key: d4-menu-lists-layer-machine
    rounds: [1]
    status: resolved
  - key: d1-rationale-wrong-staging-path
    rounds: [1]
    status: resolved
  - key: gate-language-neutral-risk-contradicts-d5
    rounds: [1]
    status: resolved
  - key: gate-load-failure-no-pair-way-forward
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1 raised ten findings; round 2's fresh review re-raised none of them and rejected the three focus points on D1, D3/D4 and D2/D5 as not defects, so all ten are resolved.
Round 2 raised one finding: the D3 gate load failure message names only steps from the prime, so a pair session reading it has no way forward and is steered towards the unguarded file removal.
Its fix adds a non-prime clause to the message, a matching line in D3's Bound and an acceptance criterion, which changes the decision's contract.
