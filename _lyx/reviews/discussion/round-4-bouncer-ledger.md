---
round: 4
ledger:
  - key: d13-tmuxkit-logger-import-cycle
    rounds: [1]
    status: resolved
  - key: d11-per-site-verdict-finish-line
    rounds: [1, 2, 3, 4]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d16-d17-dropped-landing-notice-no-recovery
    rounds: [1, 2, 3, 4]
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

Round 4 raised three findings.
The dropped-landing-notice key is still BLOCKING: D17's ready-task fallback starts any ready task at every wake with no stated bound (F1), and D16 names the wrong board state for a landed task, since Finalize marks it done and the Land row prunes it before teardown (F2).
Both findings are filed under this one key.
D11's remaining issue shrank to LOW wording: the audit's reach is stated differently in the exemption sentence, the rationale and the acceptance row (F3).
