---
round: 1
ledger:
  - key: recent-history-cap-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: comment-hard-wrap-mid-sentence
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: stale-self-report-wording-in-warns
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised three non-gating findings: F1 maps to `recent-history-cap-untested`, F2 to `comment-hard-wrap-mid-sentence`, F3 to `stale-self-report-wording-in-warns`.
None is gating, so the loop converges.
