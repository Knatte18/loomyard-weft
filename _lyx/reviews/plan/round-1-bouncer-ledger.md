---
round: 1
ledger:
  - key: card1-lockdir-error-asserted-twice
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card4-origin-weft-only-adopt-pinned-twice
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card6-uptodate-fake-row-redundant
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card3-failed-adopt-leaves-fetched-branch-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 found two duplicated-coverage findings at MEDIUM (cards 1 and 4), one at LOW (card 6), and one LOW consistency gap in card 3's attack-surface bound.
All four are open.
