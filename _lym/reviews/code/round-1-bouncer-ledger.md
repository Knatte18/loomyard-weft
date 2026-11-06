---
round: 1
ledger:
  - key: driftbaseline-engine-name-bare-literal
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: skipfileargument-eats-enclosing-quote
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: skipfileargument-breaks-backslash-continuation
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: judge-session-mark-order-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: volatile-keys-hub-only-refusal-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: expect-pass-volatile-recording-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first ledger.
F1 (BLOCKING) is the bare `DriftBaseline` literal in loomcli wiring.
F2 and F3 (MEDIUM) are the placeholder migration's quote and backslash-continuation handling in `diffmode.go`.
F4, F5 and F6 (LOW) are missing test rows for the judge-session mark order, the hub-only volatile keys, and the expect-pass recording.
