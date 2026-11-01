---
round: 1
ledger:
  - key: fork-reading-waves-miss-checklist-calls
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fork-reading-wave1-drops-at-definitions-limit
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW consistency findings on `contracts/stencils/burler/burler-fork-reading.md`.
F1 (key `fork-reading-waves-miss-checklist-calls`) says the waves name no first-wave call for callees, other-process paths or failure paths.
F2 (key `fork-reading-wave1-drops-at-definitions-limit`) says wave 1 drops the at-definitions limit on file reads.
Both fixes are local wording changes the finding text already specifies.
