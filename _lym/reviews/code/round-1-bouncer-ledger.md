---
round: 1
ledger:
  - key: parent-review-ask-strand-claim-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: merge-danger-omits-undelivered-ask-state
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: stencil-narrows-undelivered-to-creel-refusal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all open.
F1 maps to `parent-review-ask-strand-claim-untested`: no test pins the strand the `ask` verb passes to the asker.
F2 maps to `merge-danger-omits-undelivered-ask-state`: the summary's merge-danger section leaves out the new `undelivered` ask-record state.
F3 maps to `stencil-narrows-undelivered-to-creel-refusal`: the stencil and the verb's Long text say undelivered means only a creel refusal.
