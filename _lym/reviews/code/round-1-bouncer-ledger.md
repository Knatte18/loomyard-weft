---
round: 1
ledger:
  - key: stale-append-system-prompt-comments
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: burler-reading-sentence-after-fork-reading
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-creel-entry-names-only-creel-sentence
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: webster-gate-once-in-step-2
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: darn-step-4-wording
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings, none BLOCKING and none MEDIUM.
Four are stale or misplaced wording left by the batching-sentence change (comments, burler stencil placement, the creel overview entry, darn Step 4).
The fifth, the webster step 2 "once" wording, is `design` class at LOW.
Each fix is a local rewording the finding text already specifies.
