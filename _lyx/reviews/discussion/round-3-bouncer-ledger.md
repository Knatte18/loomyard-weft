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
The dropped-landing-notice key escalated to BLOCKING: D17 gates the `origin/main` versus `origin/prod` comparison on an empty deploy key, and this hub sets one, so the fallback has no durable baseline where it matters.
The proposed fix is a board-based check (done entry or gone pair means landed), with the `origin/prod` comparison kept for deploy recovery only.
D11's remaining issue is wording: the audit reach is stated as every Warn line in some places and as in-loop sites only in others.
