---
round: 1
ledger:
  - key: card16-bound-misnames-empty-params-key
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised one finding (F1): card 16's attack-surface bound says `WriteSeed` could never write the shut-out seeds, but `WriteSeed` does not refuse an empty params key.
The bound is sound in effect; only the sentence naming the reason is wrong.
It is non-gating and does not block convergence.
