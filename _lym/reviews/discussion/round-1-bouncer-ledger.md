---
round: 1
ledger:
  - key: pending-flag-respawn-pushes-code-alone
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d9-departs-from-intents-basis
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: take-hold-bound-overclaims-shellguard
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: take-hold-guard-row-text-reaches-agents
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: adopt-rerun-resets-adopt-fix-hold
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: given-up-code-tip-can-land
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: registry-lets-two-machines-claim-one-name
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-upgrade-false-premise-kept-cancels
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hold-freezes-owed-pushes-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: handed-over-pair-lingers-on-live-dobby
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: beam-status-loses-adopt-section
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-launcher-for-beam-only-machine
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-departure-from-every-reader-moves
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: already-read-takeover-mark-unnamed
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d4-handover-skips-tags-and-notice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: upgrade-notice-omits-pause-resume
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: lock-owed-push-read-as-offline
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: finalize-cut-after-local-merge-waits-out-hold
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: new-hold-caller-collides-with-holdcaller
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised nineteen findings, one BLOCKING.
The BLOCKING one (F1) is the pending-flag respawn pushing a pair's code side alone, past the records-first order.
Other themes: the hold and its interaction with owed pushes and re-runs (F5, F9, F18), the registry and takeover mechanics (F6, F7, F14), D6's `--take-hold` bound and guard text (F3, F4), and upgrade premises (F8, F16).
F2 and F13 are `decision` findings for the run's parent; the fixer does not take them.
No prior ledger exists, so nothing is carried forward and nothing is resolved.
