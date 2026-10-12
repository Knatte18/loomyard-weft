---
round: 1
ledger:
  - key: kept-values-corrupted-by-applypatch-roundtrip
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-understates-kept-key-output-sites
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-sweep-latency-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-bound-misstates-kept-key-origin
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d7-retry-clock-vs-root-now
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d7-rationale-mixed-into-decision
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised six findings, all open.
Three are gating: `ApplyPatch` re-encoding kept numbers through `float64` (F1), kept keys also printed by upsert, promote and merge (F2), and D7's per-process sweep latency left unbounded (F3).
The rest are a D4 Bound wording gap (F4), the D7 retry clock's relation to `Root.now` (F5) and D7's missing Rationale line (F6).
