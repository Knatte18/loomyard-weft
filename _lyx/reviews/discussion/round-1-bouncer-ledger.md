---
round: 1
ledger:
  - key: interrupt-marker-counted-as-working
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: orch-transcripts-in-idle-union
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: weft-default-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: span-start-fallback-escapes-clip
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: shedrun-helper-shape-open
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: restated-assumptions-rearged-alternative
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised six findings, none seen before.
The two gating ones are the interrupt marker (F1) and the unbounded weft default (F3).
The orch-transcript rule (F2) is a MEDIUM `decision` finding and does not gate.
