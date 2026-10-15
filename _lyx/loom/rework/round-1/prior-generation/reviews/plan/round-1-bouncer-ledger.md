---
round: 1
ledger:
  - key: live-ended-after-last-file-path-unclaimed
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: directive-banner-not-updated-for-optional-markers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: writer-fork-render-row-repeats-wordingclaims-phrases
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings, all local corrections to card 2 whose text the findings fix.
They are F1 (a missing wordingClaims claim for the live sub-case), F2 (the directive banner and the new part stencils' banners) and F3 (two phrases double-pinned in the render test).
No finding is gating.
