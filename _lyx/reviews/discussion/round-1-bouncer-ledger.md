---
round: 1
ledger:
  - key: d2-gate-table-precedence-and-no-pr-mode
    rounds: [1]
    status: resolved
  - key: approve-row-check-moves-to-gate
    rounds: [1]
    status: resolved
  - key: bounce-budget-boundary-and-exhausted-state
    rounds: [1]
    status: resolved
  - key: rework-append-only-baseline-and-blocked-recovery
    rounds: [1]
    status: resolved
  - key: rework-crash-reentry-mechanism
    rounds: [1]
    status: resolved
  - key: batten-decision-identity
    rounds: [1]
    status: resolved
  - key: hand-made-fix-is-decision-not-assumption
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-missing-origin-url-refusal
    rounds: [1]
    status: resolved
  - key: d6-rationale-argues-fresh-alternative
    rounds: [1]
    status: resolved
---

Round 1 raised nine findings, one blocking and five medium, all about completeness and internal consistency of what the plan writer receives.
The blocking one was the D2 gate table's unstated precedence and missing no-PR-required row; the mediums covered `approve`'s row check, the bounce-budget boundary, the rework row's append-only baseline and recovery, crash re-entry, and batten's decision identity.
Two relocations (the hand-made-fix consequence into D2, the `--fresh` clause out of D6) were legitimate under the rubric's relocation rule.
The fixer applied all nine; the judge verified each against the current pair and against `shedengine`'s bounce counting, and approved.
No finding is carried open into a later round.
