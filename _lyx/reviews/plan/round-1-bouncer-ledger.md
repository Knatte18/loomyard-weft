---
round: 1
ledger:
  - key: board-note-slug-over-32-chars
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: heading-assertion-pins-suite-text
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: removal-list-omits-s8-separator
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three consistency findings, all local corrections.
The BLOCKING one is the D6 board note slug, which exceeds the 32-character limit of `lyx board upsert`.
The LOW one is card 2's heading assertion, which pins suite text against the decision record's constraint.
The NIT is card 1's removal list missing the separator after S8.
