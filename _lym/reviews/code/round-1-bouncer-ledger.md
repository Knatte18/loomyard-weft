---
round: 1
ledger:
  - key: own-unjudged-run-makes-candidate-unconfirmed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: gateslot-doc-still-says-timeout-is-kill
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: gate-template-comments-still-say-timeout-is-kill
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: timed-out-covering-rerun-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: testoutput-line-assembly-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: empty-contained-unconfirmed-key-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 opened six findings, none seen before.
F1 changes how `JudgeMutants` and `runMutants` treat a candidate's own unjudged run, so its fix needs a fresh review.
F2 and F3 are stale prose about a timeout counting as a kill.
F4, F5 and F6 are test gaps.
