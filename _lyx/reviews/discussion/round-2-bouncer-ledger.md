---
round: 2
ledger:
  - key: d5-interim-notice-routing-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: fixer-fork-notice-path-unguarded
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: foreground-waits-meet-idle-watchdogs
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
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

Round 1 raised seven findings; round 2 raised three, all design.
The round-1 fixes for the `CLAUDE.md` line, the D6 brief wording, the D1 bound wording and the repeated gate rule drew no fresh finding.

The three open keys were each fixed in round 1 by new text, and that text is where round 2 found the faults.
D5's bound rests on a premise shuttle contradicts: a fork that sent an interim notice most likely no longer counts as outstanding, and a turn end with output files missing is held until `master_timeout_min` with a hold notice to the parent, not ended.
The fixer-fork guard fails because a gated plain Stop with both output files present is a verify-gate arrival whatever Merriam rewrites.
D9's held-call reading cannot tell a held tool call from an abandoned one, so a stopped fork reads active for the rest of the run.
