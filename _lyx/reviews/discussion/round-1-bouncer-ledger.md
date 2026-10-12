---
round: 1
ledger:
  - key: inject-unbounded-interrupt-key-path
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: still-busy-reading-vs-lagging-state
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: no-input-box-reason-misquoted
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: orch-retry-defeated-by-entry-short-circuit
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: orch-guard-refusals-silent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d1-overstates-no-drift
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised six findings, none recurring.
F1 (`inject-unbounded-interrupt-key-path`) is the only gating one: D7's rule has an unbounded `Runner.Inject` bypass.
F2 and F3 are MEDIUM consistency findings on D4/D5 and D6; F4 to F6 are LOW findings on the orch watcher retry, orch refusal visibility and D1's drift claim.
