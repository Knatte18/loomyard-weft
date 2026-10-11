---
round: 1
ledger:
  - key: auto-accept-after-fix-gate-exhausted
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: acceptance-record-archive-lifecycle
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: empty-review-skip-tolerant-parse
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
---

Round 1 raised three findings.
Auto-accept (D11) can accept a target whose fix gate failed every attempt, because the Bouncer replays an earlier CONTINUE with the budget spent.
The acceptance record's fate on each archive exit (D3, D4) is unstated.
The empty-review skip (D4) trusts a parser that tolerates unknown frontmatter keys.
