---
round: 1
ledger:
  - key: rename-mention-boundary-coverage
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rename-cli-id-addressing-untested
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: store-rename-doc-points-at-board-rename
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists.
F1 (mention boundary tests at text start, end, exact match and digit neighbours), F2 (rename by id through the CLI payload) and F3 (`Store.Rename` doc stating its own refusals) are all local corrections the finding text fixes.
None is gating.
