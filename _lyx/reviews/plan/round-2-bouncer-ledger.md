---
round: 2
ledger:
  - key: card7-landdeps-pull-literals-missed
    rounds: [1]
    status: resolved
  - key: finalize-entered-counts-current-producer
    rounds: [1]
    status: resolved
  - key: ended-way-forward-names-batten-cleanup
    rounds: [1]
    status: resolved
  - key: lsremote-deadline-needs-gitexec-change
    rounds: [1]
    status: resolved
  - key: gone-backoff-sleep-package-var
    rounds: [1]
    status: resolved
  - key: commit-pending-records-try-lock-no-home
    rounds: [1]
    status: resolved
  - key: local-remove-respawns-pair-push
    rounds: [1]
    status: resolved
  - key: publish-push-escapes-gone-rewound-checks
    rounds: [1]
    status: resolved
  - key: hubpaused-read-before-gate-wait
    rounds: [1]
    status: resolved
  - key: stencilseeding-hubscenario-not-updated
    rounds: [1]
    status: resolved
  - key: failedpushwayforward-dead-ends
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: stops-name-no-succeeding-step
    rounds: [1]
    status: resolved
  - key: fabricreplay-lacks-board-config
    rounds: [1]
    status: resolved
  - key: transport-class-no-backoff
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: ended-busy-no-way-forward
    rounds: [1]
    status: resolved
  - key: resume-landing-stops-lack-last-step
    rounds: [1]
    status: resolved
  - key: pushweft-pushwarpat-keep-pullrebase
    rounds: [1]
    status: resolved
  - key: writehubrules-no-class-carrier
    rounds: [1]
    status: resolved
  - key: card8-auth-error-bare-and-wrapped
    rounds: [1]
    status: resolved
  - key: reconcile-stops-no-way-forward
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: pushclassified-no-prime-discriminator
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: backoff-delays-unexported
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: local-remove-still-reaches-network
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: land-pull-closure-hides-warp-failure
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: failed-push-callers-repeat-stop
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: cleanup-refusal-no-reaching-test
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: pair-removals-bypass-mutation-record
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: finalize-entered-read-after-move
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised 19 findings, five BLOCKING.
Round 2 raised 11 findings, four BLOCKING, and re-raised none of the round-1 keys except two.
The way-forward defect of `failedpushwayforward-dead-ends` came back at BLOCKING for the prime's code branch (F1), and the transport handling of the gone path (F5) continues `transport-class-no-backoff`.
The other round-1 keys are carried as resolved.
Nine round-2 findings are new keys: F2, F3, F4, F6, F7, F8, F9, F10 and F11.
The open design-wrong items are the prime-aware way forward (F1), the way-forward carrier for reconcile stops (F2) and the mutation record for pair removals (F10).
