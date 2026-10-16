---
round: 1
ledger:
  - key: card2-outside-root-exemption
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: card2-scan-end-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card4-read-error-after-used-restep
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all open.
Card 2 exempts a package argument outside the worktree root without a decision licensing it, and its chain walk has scan-end branches with no named test row.
Card 4 does not state what a status read error does once the loop's one re-step is spent.
Each finding text fixes its own correction, so all three are mechanical.
