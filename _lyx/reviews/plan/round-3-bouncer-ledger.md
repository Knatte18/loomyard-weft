---
round: 3
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
    status: resolved
  - key: stops-name-no-succeeding-step
    rounds: [1]
    status: resolved
  - key: fabricreplay-lacks-board-config
    rounds: [1]
    status: resolved
  - key: transport-class-no-backoff
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: pushclassified-no-prime-discriminator
    rounds: [2]
    status: resolved
  - key: backoff-delays-unexported
    rounds: [2]
    status: resolved
  - key: local-remove-still-reaches-network
    rounds: [2]
    status: resolved
  - key: land-pull-closure-hides-warp-failure
    rounds: [2]
    status: resolved
  - key: failed-push-callers-repeat-stop
    rounds: [2]
    status: resolved
  - key: cleanup-refusal-no-reaching-test
    rounds: [2]
    status: resolved
  - key: pair-removals-bypass-mutation-record
    rounds: [2]
    status: resolved
  - key: finalize-entered-read-after-move
    rounds: [2]
    status: resolved
  - key: reconcile-mutations-not-in-envelopes
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: records-push-verb-worktree
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: way-forward-line-unasserted-at-print-hops
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: prime-flag-and-class-rows-unreached
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: two-sided-push-failure-line-duplicated
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: dirty-line-no-bolt-form
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised 19 findings, round 2 raised 11 and round 3 raised 6, so the count is falling and no design-class finding remains.
Every round-1 and round-2 key is carried as resolved: round 3 re-raised none of them.
The two keys that recurred in rounds 1 and 2, `failedpushwayforward-dead-ends` and `transport-class-no-backoff`, did not return.

Round 3 is all new keys.
The one BLOCKING item is that the reconcile's mutation record does not reach the `mutations` array of orch start, cleanup or wake (F1); its fix changes the envelope shape, so it is design-wrong and needs a fresh review.
The rest are local corrections: the worktree each way-forward command runs in (F2), the way-forward line asserted at the two printing hops (F3), rows reaching the prime flag and the Finalize and Publish classes (F4), a both-sides failure printing one line twice (F5) and a Bolt form for the dirty line (F6).
