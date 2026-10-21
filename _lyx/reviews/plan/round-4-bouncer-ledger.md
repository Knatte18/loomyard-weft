---
round: 4
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
    status: resolved
  - key: records-push-verb-worktree
    rounds: [3]
    status: resolved
  - key: way-forward-line-unasserted-at-print-hops
    rounds: [3, 4]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: prime-flag-and-class-rows-unreached
    rounds: [3]
    status: resolved
  - key: two-sided-push-failure-line-duplicated
    rounds: [3]
    status: resolved
  - key: dirty-line-no-bolt-form
    rounds: [3]
    status: resolved
  - key: rewind-keep-local-merge-noop
    rounds: [4]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: tag-created-no-mutation-kind
    rounds: [4]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: way-forward-lines-no-directory
    rounds: [4]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: way-forward-reentry-verb-shared-string
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: failed-push-mail-text-unasserted
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: publish-not-fast-forward-skips-class-line
    rounds: [4]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: orch-start-error-envelope-bare
    rounds: [4]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: reconcile-verb-two-way-forward-lines
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Rounds 1 to 3 raised 19, 11 and 6 findings; round 4 raised 9.
Every round-3 key except `way-forward-line-unasserted-at-print-hops` is carried as resolved: round 4 rejected the prime-flag, two-sided-printing, Bolt dirty-form and records-push-verb rows as covered, and its envelope findings are about new gaps (the error envelope, the tag kind) under new keys.

Round 4 is mostly new keys.
The BLOCKING item is the keep-local rewind choice, whose `ours` merge changes nothing and whose follow-up push card 9's hold-back refuses; its fix rewrites the prescribed step, so it is design-wrong.
The other design finding splits `BranchOutcome.WayForward` into a repair step and a per-hop re-entry, also design-wrong.
`way-forward-line-unasserted-at-print-hops` is reopened for orch start, batten resume, Merge-In and Finalize, which assert a named stop rather than the judge's exact line.
The remaining items are local corrections: a `tag_created` kind and its rows, the directory in the dirty and rewritten lines, the mail text row, Publish's `not_fast_forward` line, the orch start error envelope and a doubled line in the reconcile verb.
