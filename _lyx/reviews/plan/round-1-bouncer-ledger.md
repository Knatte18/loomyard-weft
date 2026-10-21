---
round: 1
ledger:
  - key: step-recreates-weft-torn-down-mid-repair
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: ways-forward-omit-target-path
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: doc-comments-still-claim-every-junction-swept
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hook-seed-order-against-template-build-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: failed-exclude-row-needs-existing-lock-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: case-3-parent-mkdirall-failure-has-no-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: seam-setter-location-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: card-3-unused-uses-entry
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eight findings, all LOW or NIT, none gating.
F1 asks for a guard against recreating a weft torn down mid-repair, a behavior change, so it is marked `design-wrong`.
The other seven are local corrections whose text the finding fixes.
