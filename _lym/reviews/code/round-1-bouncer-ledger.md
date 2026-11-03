---
round: 1
ledger:
  - key: prunable-registration-skips-absent-pair-judgment
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: battencli-duplicates-archive-tag-format
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: adopt-only-head-and-fork-point-skips-unpinned
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: error-branches-without-test-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings.
F1 (design, MEDIUM) maps to the prunable-registration key and gates.
F2 to F4 are LOW consistency findings; each fix text fixes its result, so all four are mechanical.
