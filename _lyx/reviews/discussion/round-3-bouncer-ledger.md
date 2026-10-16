---
round: 3
ledger:
  - key: d13-tmuxkit-logger-import-cycle
    rounds: [1]
    status: resolved
  - key: d11-per-site-verdict-finish-line
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d16-d17-dropped-landing-notice-no-recovery
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d5-reason-file-currency-and-dedupe-key
    rounds: [1]
    status: resolved
  - key: record-forensic-narrative-relocation
    rounds: [1]
    status: resolved
---

Round 3 raised two findings, both on keys carried from earlier rounds.
The dropped-landing-notice gap is now BLOCKING: the `origin/prod` baseline fails on a hub whose landing deploy key is set, so the orch needs a board-based check that does not depend on the deploy key.
D11's audit reach is still worded three ways; the fix is to state the narrow reading (Info-or-above lines, Warn included, in a loop or polled path) everywhere.
