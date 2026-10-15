---
round: 1
ledger:
  - key: d8-carrier-task-summary-false-premise
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pattern-read-set-three-wordings
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: empty-pattern-directive-criterion-unmeetable
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d3-replaced-span-ambiguous
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d1-read-ban-bound-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open.
The one gating finding is that D8 names the task summary as the carrier of the post-landing re-measurement, which the Master writes only from batch digests; the fix changes the carrier to an orch board note, so it needs a fresh review.
The other four are local wording corrections whose result each finding text already fixes: the three wordings of the pattern read set, the unmeetable empty-directive acceptance criterion, D3's ambiguous replaced span, and the unstated bound on D1's prompt-only read ban.
