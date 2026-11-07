---
round: 1
ledger:
  - key: clone-reorder-unproven-render
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: walk-failure-drops-seam-warning
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: render-warnings-only-logged
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: made-event-position-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: clone-envelope-seam-warning-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised five design MEDIUM findings, F1 to F5, mapped in order to the keys above.
F1 asks the fixer to settle which cause leaves the prime unrendered, so its fix may change the approach and needs a fresh review.
The other four are local corrections whose result the finding text fixes.
There was no previous ledger.
