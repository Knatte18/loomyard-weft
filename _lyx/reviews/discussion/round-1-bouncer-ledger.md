---
round: 1
ledger:
  - key: d5-interim-notice-routing-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: fixer-fork-notice-path-unguarded
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: foreground-waits-meet-idle-watchdogs
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: claude-md-still-says-background-gate
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: d6-brief-content-ambiguous
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d1-bound-vs-open-risk-ceiling-outcome
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: gate-rule-stated-in-several-places
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised seven findings against the decision record, none seen before.
Two are gating design findings: D5's interim-notice routing edge has no stated bound, and foreground gate waits of up to an hour meet the step idle kill and batten's quiet notice without a decided remedy.
The rest are scope gaps (fixer fork path, `CLAUDE.md` line 31), an ambiguous brief instruction in D6, and two consistency nits in D1 and D2 to D4.
