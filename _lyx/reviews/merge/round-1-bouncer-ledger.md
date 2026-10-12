---
round: 1
ledger:
  - key: parseseedtolerant-duplicates-decodeseed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised one finding, F1: after the parent made `decodeSeed` tolerant, `ParseSeedTolerant` repeats its decode body.
It breaks no invariant and gates nothing.
