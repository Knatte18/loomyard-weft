---
round: 1
ledger:
  - key: d13-tmuxkit-logger-import-cycle
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d11-per-site-verdict-finish-line
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d16-d17-dropped-landing-notice-no-recovery
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d5-reason-file-currency-and-dedupe-key
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: record-forensic-narrative-relocation
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings.
The gating pair is the D13 import cycle (the logger should discard its own stderr under `testing.Testing()`) and D11's unbounded per-site verdict list.
D16/D17 needs a next-wake fallback for dropped landing notices.
D5 needs a reason-file write rule and a caller-supplied dedupe key.
The NIT moves forensic narrative from the decision record to the support log.
