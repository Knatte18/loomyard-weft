---
round: 1
ledger:
  - key: card7-landdeps-pull-literals-missed
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: finalize-entered-counts-current-producer
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: ended-way-forward-names-batten-cleanup
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: lsremote-deadline-needs-gitexec-change
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: gone-backoff-sleep-package-var
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: commit-pending-records-try-lock-no-home
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: local-remove-respawns-pair-push
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: publish-push-escapes-gone-rewound-checks
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: hubpaused-read-before-gate-wait
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stencilseeding-hubscenario-not-updated
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: failedpushwayforward-dead-ends
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stops-name-no-succeeding-step
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fabricreplay-lacks-board-config
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: transport-class-no-backoff
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: ended-busy-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resume-landing-stops-lack-last-step
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pushweft-pushwarpat-keep-pullrebase
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: writehubrules-no-class-carrier
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card8-auth-error-bare-and-wrapped
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 (first round, no prior ledger) raised 19 findings: five BLOCKING (F1 to F5), eight MEDIUM, five LOW, one NIT.
The BLOCKING ones: a missed `LandDeps.Pull` literal in card 7 (F1), a `finalizeEntered` that makes D8's Finalize check dead (F2), an ended way forward that removes no pair (F3), a per-attempt ls-remote deadline needing an untargeted `gitexec` change (F4), and a package-level sleep var against `PATTERN-test-speed` (F5).
Design-class MEDIUMs: F7, F9, F11, F12, F13.
Findings map to keys in order: F1 to F19.
All are open.
