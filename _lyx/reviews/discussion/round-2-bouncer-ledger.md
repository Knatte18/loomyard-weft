---
round: 2
ledger:
  - key: kept-values-corrupted-by-applypatch-roundtrip
    rounds: [1]
    status: resolved
  - key: d4-understates-kept-key-output-sites
    rounds: [1]
    status: resolved
  - key: d7-sweep-latency-unbounded
    rounds: [1]
    status: resolved
  - key: d4-bound-misstates-kept-key-origin
    rounds: [1]
    status: resolved
  - key: d7-retry-clock-vs-root-now
    rounds: [1]
    status: resolved
  - key: d7-rationale-mixed-into-decision
    rounds: [1]
    status: resolved
  - key: d1-byte-for-byte-unattainable
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-bound-misattributes-one-second-path
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised six findings; round 2 did not re-raise any, so all six are resolved.
Round 2 raised two new findings: D1's byte-for-byte requirement is unattainable through the indented encoder (MEDIUM), and D7's Bound misattributes when the one-second path is reached (LOW).
