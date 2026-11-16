---
round: 2
ledger:
  - key: pending-flag-respawn-pushes-code-alone
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d9-departs-from-intents-basis
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: take-hold-bound-overclaims-shellguard
    rounds: [1]
    status: resolved
  - key: take-hold-guard-row-text-reaches-agents
    rounds: [1]
    status: resolved
  - key: adopt-rerun-resets-adopt-fix-hold
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: given-up-code-tip-can-land
    rounds: [1]
    status: resolved
  - key: registry-lets-two-machines-claim-one-name
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-upgrade-false-premise-kept-cancels
    rounds: [1]
    status: resolved
  - key: hold-freezes-owed-pushes-unstated
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: handed-over-pair-lingers-on-live-dobby
    rounds: [1]
    status: resolved
  - key: beam-status-loses-adopt-section
    rounds: [1]
    status: resolved
  - key: no-launcher-for-beam-only-machine
    rounds: [1]
    status: resolved
  - key: d8-departure-from-every-reader-moves
    rounds: [1]
    status: resolved
  - key: already-read-takeover-mark-unnamed
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d4-handover-skips-tags-and-notice
    rounds: [1]
    status: resolved
  - key: upgrade-notice-omits-pause-resume
    rounds: [1]
    status: resolved
  - key: lock-owed-push-read-as-offline
    rounds: [1]
    status: resolved
  - key: finalize-cut-after-local-merge-waits-out-hold
    rounds: [1]
    status: resolved
  - key: new-hold-caller-collides-with-holdcaller
    rounds: [1]
    status: resolved
  - key: bolt-hold-read-as-operator-pause
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: machine-id-has-no-lifetime
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: forget-former-hands-records-to-another-machine
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: takeover-read-cannot-see-former-names
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: upgrade-first-read-has-no-order
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d9-pushed-state-claim-misses-owed-push
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: code-branch-criterion-contradicts-handover
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-runfsm-exception-too-narrow
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: adopt-check-left-unstated
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: orphan-first-sight-time-has-no-home
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: partly-overlapping-hold-subjects-undefined
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1's nineteen keys: eleven are not raised again in round 2 and are resolved.
Five carry into round 2 as open:
the detached pair push that keeps code before records (round 2 F1, the same key as round 1's pending-flag respawn, now through a different holder),
the registry name comparison (F3), the owed-push ordering under a hold (F7), the plain re-run's reset (F8),
and the hand-over mark, whose root is still unstated (F11).
The key `d9-departs-from-intents-basis` carries as the leftover Open risks line the parent's entry already settles (F9).

Round 2 raised twelve new keys.
The BLOCKING one is `bolt-hold-read-as-operator-pause` (F2): `pauserecord`'s readers word a hold as an operator pause and name `lym tender resume`.
The registry and machine-name work draws most of the MEDIUM findings (F3 to F6).
The rest are LOW and mostly local: unstated orders, a stale criterion, a narrow Scope exception, and three undefined marks or sets.
