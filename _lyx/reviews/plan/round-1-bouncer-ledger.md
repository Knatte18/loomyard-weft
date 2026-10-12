---
round: 1
ledger:
  - key: card2-outside-root-exemption
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: card2-scan-end-branches-unreached
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card4-read-error-after-spent-restep
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised three findings against the plan.
F1 (card 2 exempts arguments outside the worktree root without a decision) and F3 (card 4 is silent on a read error after the re-step is spent) are non-gating.
F2 (card 2's copied chain walk has scan-end branches no row reaches) is design-class MEDIUM and keeps the loop open.
