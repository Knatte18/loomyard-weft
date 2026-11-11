---
round: 1
ledger:
  - key: board-skill-missing-promote-depends-on
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: promote-sibling-notice-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: promote-depends-on-edge-paths-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 opened three findings, none carried from earlier rounds.
The board skill's promote section lacks the new `depends_on` guidance.
The sibling notice that `promote` returns has no CLI or engine test.
The edge paths of promote's `depends_on` (equal list, empty list, null, non-string element) have no rows.
Each fix is a local addition whose content the finding text states, so all three are mechanical.
