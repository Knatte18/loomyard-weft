---
round: 1
ledger:
  - key: attempt-key-vs-same-stop-rule-error-stop
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
  - key: unpause-overrides-human-pause
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-stencil-only-bound-states-no-consequence
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: missing-watched-marker-admits-arbiter-at-publish
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: scope-points-at-missing-d14
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: negative-timeout-vs-negative-budget
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: batten-hold-release-paths-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised seven findings, all open.
The gating ones are the error-stop key collision between D1 and D7 (BLOCKING), the un-pause overriding a human pause in D6, the missing consequence statement in D2's stencil-only bound, and the missing-marker gap at the batten-owned Publish halt.
Three LOW findings cover a dangling D14 reference, the negative-timeout handling, and the batten hold's untested release paths.
