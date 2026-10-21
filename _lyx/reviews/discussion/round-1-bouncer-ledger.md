---
round: 1
ledger:
  - key: machine-layer-anchor-without-lyx
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-message-reaches-board-only-checks
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: retired-machine-key-two-outcomes
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d3-from-prime-and-removal-hatch-bound
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: every-hub-wide-module-gains-machine-layer
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: editor-save-outcome-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d2-bound-misstates-direct-write
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d4-menu-lists-layer-machine
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-rationale-wrong-staging-path
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gate-language-neutral-risk-contradicts-d5
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised ten findings, all open.
The gating one is the D1 applicability rule for a worktree whose anchor has no `_lyx/`: judged on `load`'s cleared dirs the machine layer drops, judged on the told dirs it stays.
The three other MEDIUM findings concern the reach of D3's failure message, the two stated outcomes for a retired machine key, and the missing "from the prime" qualifier with an unbounded removal hatch.
The LOW findings tighten D5's scope to every hub-wide module, the editor-save outcome, D2's bound, the menu's `--layer` claim, a wrong staging path in D1's rationale, and the `gate-language-neutral` risk.
