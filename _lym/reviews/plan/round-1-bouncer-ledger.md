---
round: 1
ledger:
  - key: dobby-hand-over-trigger-reowes-catch-up
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: tender-resume-refuses-on-held-before-hold-rules
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: cards-8-23-unlisted-tests-on-old-seams
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: card-16-held-refusal-shadowed-by-refuse-by-act
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: dobby-start-registration-refusal-no-notice-no-status
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: machine-names-of-reads-registered-name-as-own
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: parent-lock-exemption-from-bolt-hold-uncarried
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: card-19-commit-run-intent-resign-arrow
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card-17-forced-pair-removal-no-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: owner-pause-text-stays-after-card-23
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card-11-pattern-cli-cobra-machinecli-line
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: absences-decision-false-premise
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 is the first round and has no earlier ledger.
Twelve findings are open, keyed one to one with F1 through F12 of the round-1 review:
F1 to `dobby-hand-over-trigger-reowes-catch-up`, F2 to `tender-resume-refuses-on-held-before-hold-rules`,
F3 to `cards-8-23-unlisted-tests-on-old-seams`, F4 to `card-16-held-refusal-shadowed-by-refuse-by-act`,
F5 to `dobby-start-registration-refusal-no-notice-no-status`, F6 to `machine-names-of-reads-registered-name-as-own`,
F7 to `parent-lock-exemption-from-bolt-hold-uncarried`, F8 to `card-19-commit-run-intent-resign-arrow`,
F9 to `card-17-forced-pair-removal-no-bound`, F10 to `owner-pause-text-stays-after-card-23`,
F11 to `card-11-pattern-cli-cobra-machinecli-line`, F12 to `absences-decision-false-premise`.
Three are BLOCKING; the rest are MEDIUM or LOW.
Fixes that change behavior or a decision are marked `design-wrong`, so a fresh review sees them; target and wording fixes are `mechanical`.
