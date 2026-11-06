---
round: 1
ledger:
  - key: stencil-prose-asserted-in-tests
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: assist-record-keys-stale-across-attempts
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: fan-wiring-to-keep-assist-reports-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: returned-searches-whole-file
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-every-assistant-comments
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: duplicate-fan-output-assertions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, F1 to F6, mapped in order to the keys above.
F1 is BLOCKING for asserting stencil prose against `PATTERN-test-speed`.
F2 and F3 are the gating MEDIUM design findings.
F4 to F6 are LOW.
Each fix is a local correction its finding text already specifies.
