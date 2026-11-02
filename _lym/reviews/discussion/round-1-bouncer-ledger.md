---
round: 1
ledger:
  - key: darn-write-restart-halts-on-clean-entry
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: expired-probe-reads-terminal-records
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-deadline-overrun-bound-understated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-join-rule-fails-after-rerun
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: killed-card-runner-second-fixer
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-plan-write-lens-validator-loop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: plan-write-fixer-lens-review-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-unattended-edits-no-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: webster-fork-text-unconditional
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d4-caller-tests-not-grounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card-fixer-count-rests-on-prompt-cap
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: runproc-registration-changes-webster-fixer
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fixer-timeout-unresearched
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: removed-citations-lack-mark
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists.
Keys map to review findings F1 to F14 in order.
F1 is BLOCKING: a restarted Darn-Write driver halts on the clean entry before it can wait on a live fixer.
F2 to F8 are MEDIUM design findings on the probe, the deadline bound, the join rule, the killed runner, the Plan-Write lens path, the lens-review record and the unattended-edit bound.
F9 to F14 are the MEDIUM and LOW consistency and bound-wording findings.
