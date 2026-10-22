---
round: 1
ledger:
  - key: fallback-misses-permission-denied-journal
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pause-record-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-sleepjournal-file-header
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings.
F1 is the gating one: `journalctl` exits 0 with empty stdout when the user lacks journal access, so `Read` reports a readable journal with no sleep and the hub-pause fallback never runs.
Its fix changes the `Read` contract by passing stderr in, so it needs a fresh review.
F2 asks for a stronger assertion in the existing integration check, and F3 is a one-line header edit; both are local corrections.
