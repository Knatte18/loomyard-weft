---
round: 2
ledger:
  - key: wired-stencils-check-never-shown-to-refuse
    rounds: [1]
    status: resolved
  - key: checkmarkers-counts-blank-values-as-filling
    rounds: [1]
    status: resolved
  - key: table-build-failure-shown-as-unfillable-stencil
    rounds: [1]
    status: resolved
  - key: preflightentry-forwarding-unpinned
    rounds: [1]
    status: resolved
  - key: lead-role-label-pinned-only-on-validate
    rounds: [1]
    status: resolved
  - key: checkmarkers-branches-without-a-row
    rounds: [1]
    status: resolved
  - key: stale-preflight-file-header
    rounds: [1]
    status: resolved
  - key: preflight-checks-only-darn-write-table
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: checkmarkers-refusal-blames-skew-for-blank-value
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, one MEDIUM and six LOW.
Round 2's review answers each of their focus questions as not a defect, so all seven are resolved.
Round 2 raised two new LOW findings: the Preflight stencils check covers only the Darn-Write table, and the CheckMarkers refusal names version skew alone when the cause is a blank value.
Both fixes are local and their text names the change, so both are marked mechanical.
