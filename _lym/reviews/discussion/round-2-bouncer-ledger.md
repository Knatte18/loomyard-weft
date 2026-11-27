---
round: 2
ledger:
  - key: arbiter-hard-cap-lands-open-blocking-unattended
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: arbiter-budget-accept-changes-level-false
    rounds: [1]
    status: resolved
  - key: severe-halt-reaches-no-one-under-true
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: chase-misses-question-and-dead-driver-stalls
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: table-has-no-verdict-at-false
    rounds: [1]
    status: resolved
  - key: rows-without-operator-cell-lose-third-halt-rule
    rounds: [1]
    status: resolved
  - key: orch-classified-rows-table-vs-relay-list
    rounds: [1]
    status: resolved
  - key: halt-count-undefined-on-existing-records
    rounds: [1]
    status: resolved
  - key: arbiter-continue-once-has-no-memory
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pause-resume-rests-on-missing-line
    rounds: [1]
    status: resolved
  - key: lift-resume-undoes-orch-hold
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: batten-child-start-would-block
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: run-lock-branch-keeps-looping-refusal
    rounds: [1]
    status: resolved
  - key: stop-stamps-have-no-named-home
    rounds: [1]
    status: resolved
  - key: permission-dialog-has-no-notice-signal
    rounds: [1]
    status: resolved
  - key: mention-dedup-silences-second-severe-halt
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: reply-probe-answers-for-unread-reply
    rounds: [1]
    status: resolved
  - key: hung-command-class-dropped
    rounds: [1]
    status: resolved
  - key: criteria-missing-test-tiers
    rounds: [1]
    status: resolved
  - key: repeating-stop-selfreport-unassigned
    rounds: [1]
    status: resolved
  - key: double-severe-post-under-unattended
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: dead-driver-bound-not-in-table
    rounds: [1]
    status: resolved
  - key: new-driver-rows-name-no-identifying-field
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: plan-note-wrong-settle-records-file
    rounds: [1]
    status: resolved
  - key: open-risks-stale-row-name
    rounds: [1]
    status: resolved
  - key: tackler-resume-counted-more-than-once
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: arbiter-budget-accept-over-unread-input
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1's findings cluster in the arbiter's accept and continue rules, in whether a stop reaches a person,
in the stop table's coverage, and in premises about existing records, probes and verbs.

Round 2 resolved most of those keys.
The ones still open are narrower remainders in the same areas: the unattended accept rule at Finalize,
severe posts and reminders for stalls, the arbiter's spent continue, the hold and lift resume,
dedup of severe posts, and the bounce-budget row's mark.

Two keys are new in round 2, both BLOCKING premises about existing code:
how a tackler resume is recorded, and what causes a `budget` escalation.
