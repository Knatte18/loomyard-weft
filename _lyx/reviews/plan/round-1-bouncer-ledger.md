---
round: 1
ledger:
  - key: doc-go-replacement-span-miscounted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: locate-line-ambiguous-inline-code
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: multipart-row-subsumed-needles
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first round; there is no prior ledger.
F1 (LOW) is the `doc.go` span count in card 01's Intent.
F2 (NIT) is the ambiguous inline-code form of the locate fragment's replacement line.
F3 (NIT) is the multipart render row keeping subsumed needles.
All three are `consistency` findings; none gates.
