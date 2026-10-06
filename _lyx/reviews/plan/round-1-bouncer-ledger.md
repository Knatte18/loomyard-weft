---
round: 1
ledger:
  - key: card3-prunes-testtiming
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: skill-loading-carveout-uncarried
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: card18-testtiming-in-scope
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card17-catchall-packages
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card5-undefined-scenario-merge-ref
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: decision13-bounds-uncarried
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card18-link-integrity-missing
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 is the first review of the plan.
Two BLOCKING findings: card 3 still prunes `cmd/testtiming`, which belongs to core, and the skill-loading carve-out for skills-in-one-turn is carried by no card or Shared Decision.
Two MEDIUM findings: card 18 treats `cmd/testtiming` as an in-scope package for the keep check and splice, and card 17 is a 21-package catch-all.
Three LOW findings: card 5 cites an undefined `scenario-merge` decision, two Decision 13 bounds are missing from the prune procedure, and card 18 omits the link-integrity constraint.
