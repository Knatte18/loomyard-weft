---
round: 3
ledger:
  - key: d5-interim-notice-routing-unbounded
    rounds: [1, 2]
    status: resolved
  - key: fixer-fork-notice-path-unguarded
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: foreground-waits-meet-idle-watchdogs
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-shuttle-activity-contract-undocumented
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: claude-md-still-says-background-gate
    rounds: [1]
    status: resolved
  - key: d6-brief-content-ambiguous
    rounds: [1]
    status: resolved
  - key: d1-bound-vs-open-risk-ceiling-outcome
    rounds: [1]
    status: resolved
  - key: gate-rule-stated-in-several-places
    rounds: [1]
    status: resolved
---

Round 1 raised seven findings, round 2 three, round 3 three.
The round-1 keys on the `CLAUDE.md` line, the D6 brief, the D1 bound and the repeated gate rule have drawn nothing since.

`d5-interim-notice-routing-unbounded` is resolved: round 3 confirms an interim-notified fork is not outstanding and the batch-loop outcome is stated correctly.
Two design keys have now survived three rounds, each re-raised on text added to fix it.
The fixer bound in D5 still misdescribes what an arrival does: instant dirty-tree failure, a rewritten report, and strand removal with the fixer live.
D9's held-call rule still keys on the newest transcript entry, which a parallel sibling's result defeats.
The new LOW key is that D9 alters the documented `shuttleengine` activity contract without the Scope listing it.
