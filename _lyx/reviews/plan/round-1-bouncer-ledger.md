---
round: 1
ledger:
  - key: card3-departs-d2-merge-in-progress-way-forward
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: card3-realhub-checkout-path-no-evalsymlinks
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card3-form3-reaching-state-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-git-spawning-row-claimed-no-git
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: card4-parentheader-not-an-edit-target
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; there is no earlier ledger.
Five findings, none gating.
The one MEDIUM concerns a wording departure from D2 that needs a decision-record entry through the parent.
The rest are local corrections to card 3, card 4 and card 5 text.
