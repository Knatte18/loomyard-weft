---
round: 2
ledger:
  - key: darn-write-restart-halts-on-clean-entry
    rounds: [1]
    status: resolved
  - key: expired-probe-reads-terminal-records
    rounds: [1]
    status: resolved
  - key: d2-deadline-overrun-bound-understated
    rounds: [1]
    status: resolved
  - key: d4-join-rule-fails-after-rerun
    rounds: [1]
    status: resolved
  - key: killed-card-runner-second-fixer
    rounds: [1]
    status: resolved
  - key: d7-plan-write-lens-validator-loop
    rounds: [1]
    status: resolved
  - key: plan-write-fixer-lens-review-record
    rounds: [1]
    status: resolved
  - key: d5-unattended-edits-no-bound
    rounds: [1]
    status: resolved
  - key: webster-fork-text-unconditional
    rounds: [1]
    status: resolved
  - key: d4-caller-tests-not-grounded
    rounds: [1]
    status: resolved
  - key: card-fixer-count-rests-on-prompt-cap
    rounds: [1]
    status: resolved
  - key: runproc-registration-changes-webster-fixer
    rounds: [1]
    status: resolved
  - key: fixer-timeout-unresearched
    rounds: [1]
    status: resolved
  - key: removed-citations-lack-mark
    rounds: [1]
    status: resolved
  - key: finalize-off-idle-boundary-spawns-fixer
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d8-row-visit-identity-undefined
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: goto-does-not-stop-live-fixer
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-role-guard-teaches-gate-webster-role
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: fixer-field-unbounded-on-gate-keys
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: card-fixer-runproc-owner-unnamed
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card-fixer-parent-directive-unnamed
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-lens-staleness-one-writer-only
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 14 findings, led by a BLOCKING restart halt on Darn-Write's clean entry.
Round 2 raises none of those keys again, so all are resolved.

Round 2 keys map to F1 to F8 in order.
F1 is BLOCKING: a finalize evaluation reached off the writer's idle boundary, such as the deadline finalize, spawns a fixer while the writer still edits.
F2 and F3 are MEDIUM design findings: the row-visit identity that keys D8's record set, and goto leaving a live fixer running.
F4 to F8 are consistency and LOW findings on the role guard's owner, the `fixer` field's bound, the card fixer's runproc owner and parent directive, and D7's staleness wording.
