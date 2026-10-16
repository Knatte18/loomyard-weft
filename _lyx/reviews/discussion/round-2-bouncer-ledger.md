---
round: 2
ledger:
  - key: d13-tmuxkit-logger-import-cycle
    rounds: [1]
    status: resolved
  - key: d11-per-site-verdict-finish-line
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d16-d17-dropped-landing-notice-no-recovery
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-reason-file-currency-and-dedupe-key
    rounds: [1]
    status: resolved
  - key: record-forensic-narrative-relocation
    rounds: [1]
    status: resolved
---

Round 1 raised five findings; round 2 raised two.
The D13 import-cycle and D5 dedupe concerns were checked and rejected in round 2, and the forensic-narrative NIT did not recur.
D11's finish line is bounded, but its opening sentence and the Scope bullet still promise a check on every Warn site; the fix narrows the stated reach.
D17's next-wake fallback names no durable baseline; the hub notice log's landed lines, read against the last `update-plugins.sh` run, or the installed build's revision against main's HEAD, would supply one.
