---
round: 3
ledger:
  - key: attempt-key-vs-same-stop-rule-error-stop
    rounds: [1]
    status: resolved
  - key: unpause-overrides-human-pause
    rounds: [1, 2]
    status: resolved
  - key: d2-stencil-only-bound-states-no-consequence
    rounds: [1]
    status: resolved
  - key: missing-watched-marker-admits-arbiter-at-publish
    rounds: [1]
    status: resolved
  - key: scope-points-at-missing-d14
    rounds: [1]
    status: resolved
  - key: negative-timeout-vs-negative-budget
    rounds: [1]
    status: resolved
  - key: batten-hold-release-paths-unpinned
    rounds: [1]
    status: resolved
  - key: d6-unpause-clause-false-premise
    rounds: [2]
    status: resolved
  - key: applied-mark-cannot-tell-recurred-from-unstepped
    rounds: [2]
    status: resolved
  - key: d2-inputs-omit-verify-failure-record
    rounds: [2]
    status: resolved
  - key: regoto-gap-erases-plain-pause
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-busy-example-presumes-lock-order
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: rejected-alternative-contradicts-chosen-design
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised seven findings, round 2 four, round 3 three.
Every round-1 and round-2 key is resolved: round 3's review lists the pause-erasure, un-pause-clause and D2-input questions as settled.

The human-pause key narrowed over the rounds from the arbiter's goto erasing pause conditions, to a false un-pause premise, to a residual gap in round 3.
That gap is F1: the re-launch's goto still clears `pause_requested`, and D6's status-lock span cannot hold across the step.
It is a new key and goes to Open risks.
F2 notes that D7's busy-refusal example needs an ordering D6 does not fix.
F3 notes that the support log's recommend-only rejection contradicts the chosen design.
