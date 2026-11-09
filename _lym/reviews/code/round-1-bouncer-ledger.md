---
round: 1
ledger:
  - key: test-economy-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: splice-stale-report-header
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: writemap-buildfailure-replace-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: judgepackage-contained-and-kept-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: runalone-skip-with-profile-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: keyblocks-multifile-generic-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised six LOW findings: two `consistency` (F1 doc, F2 report header) and four `design` test-coverage gaps (F3 to F6).
All fixes are local corrections the finding text already specifies.
No finding is gating, so the loop converges.
