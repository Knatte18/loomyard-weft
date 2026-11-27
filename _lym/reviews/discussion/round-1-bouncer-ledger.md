---
round: 1
ledger:
  - key: arbiter-hard-cap-lands-open-blocking-unattended
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: arbiter-budget-accept-changes-level-false
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: severe-halt-reaches-no-one-under-true
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: chase-misses-question-and-dead-driver-stalls
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: table-has-no-verdict-at-false
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: rows-without-operator-cell-lose-third-halt-rule
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: orch-classified-rows-table-vs-relay-list
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: halt-count-undefined-on-existing-records
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: arbiter-continue-once-has-no-memory
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pause-resume-rests-on-missing-line
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: lift-resume-undoes-orch-hold
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: batten-child-start-would-block
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: run-lock-branch-keeps-looping-refusal
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stop-stamps-have-no-named-home
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: permission-dialog-has-no-notice-signal
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: mention-dedup-silences-second-severe-halt
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reply-probe-answers-for-unread-reply
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hung-command-class-dropped
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: criteria-missing-test-tiers
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: repeating-stop-selfreport-unassigned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: double-severe-post-under-unattended
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: dead-driver-bound-not-in-table
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: new-driver-rows-name-no-identifying-field
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: plan-note-wrong-settle-records-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: open-risks-stale-row-name
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so every key is new and open.
The findings cluster in these areas:

- the arbiter's accept and continue rules across the autonomy levels;
- whether a severe or unanswered stop reaches a person;
- the stop table's coverage of levels and classes;
- design premises that rest on existing records, probes and verbs.
