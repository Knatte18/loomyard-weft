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

Round 3 raised two findings.
F1 is the dropped-landing-notice fallback: `origin/main` versus `origin/prod` cannot reveal a drop once the Land row deploys, so the orch needs a board-done check that ignores the deploy key.
F2 is D11's audit reach, still worded three ways after the round-2 fix.
The D13, D5 and forensic-narrative keys stay resolved.
