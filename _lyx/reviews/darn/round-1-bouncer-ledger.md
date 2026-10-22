---
round: 1
ledger:
  - key: exact-join-under-pair-opening-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: floor-on-later-page-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: add-commit-subject-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three test-coverage findings on the pair-opening floor in the report's landing walk.
F1 (MEDIUM) maps to `exact-join-under-pair-opening-unpinned`, F2 (LOW) to `floor-on-later-page-unpinned`, F3 (LOW) to `add-commit-subject-unasserted`.
Each fix names the existing test row to extend, so all three are mechanical.
