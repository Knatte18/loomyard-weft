---
round: 2
ledger:
  - key: attempt-key-vs-same-stop-rule-error-stop
    rounds: [1]
    status: resolved
  - key: unpause-overrides-human-pause
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
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
    status: open
    class: design
    severity: MEDIUM
  - key: applied-mark-cannot-tell-recurred-from-unstepped
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: d2-inputs-omit-verify-failure-record
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised seven findings.
Round 2 raised four, and none of the round-1 keys except the human-pause one recurs.

The human-pause key escalated from MEDIUM to BLOCKING: the arbiter's goto clears `pause_requested`, `pause_before` and `pause_after`, so D6's state check passes after a human pause was erased.
The review's fix moves the goto into the Go re-launch, under the status lock, so the check runs first.
The round-1 error-stop key collision is resolved; its remaining gap is the new LOW key about an applied mark that cannot tell a recurred stop from a re-launch that never stepped.
D6's un-pause clause rests on a mechanism `lyx loom resume` does not have (MEDIUM, new).
A NIT notes that D2's list of Go inputs omits the Publish row's verify-failure record.
