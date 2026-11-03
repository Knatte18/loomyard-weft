---
round: 1
ledger:
  - key: wired-stencils-check-never-shown-to-refuse
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: checkmarkers-counts-blank-values-as-filling
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: table-build-failure-shown-as-unfillable-stencil
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: preflightentry-forwarding-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: lead-role-label-pinned-only-on-validate
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: checkmarkers-branches-without-a-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-preflight-file-header
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings: F1 (MEDIUM, wiring test never shows the wired stencils check refusing), F2 to F6 (LOW, design) and F7 (LOW, consistency, stale file header).
F2 to F6 and F7 are local corrections whose fix text names the change, so all seven are marked mechanical.
