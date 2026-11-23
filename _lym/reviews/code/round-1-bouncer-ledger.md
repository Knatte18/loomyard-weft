---
round: 1
ledger:
  - key: probe-refused-from-nonorch-strand-pane
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: quiet-probe-release-ends-run-mid-turn
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: reused-strand-name-inherits-open-question
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card-fixer-runner-has-no-reply-probe
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: release-hold-clear-unpinned-for-continuing-gate
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: awaitsreply-test-misses-returned-and-foreign-mail
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: hold-pause-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: setreplyprobe-wiring-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: merge-danger-blast-radius-understated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings, F1 to F9, in the order of the ledger above.
F1 and F2 are BLOCKING and decide the round: the probe signs as a program and is refused from a non-orch strand pane, and the two-quiet-probe release may end a run mid-turn if Claude's `asyncRewake` wake writes no turn start.
F3 is the MEDIUM design finding about a reused strand name inheriting an earlier session's open question.
F4 is the MEDIUM scope finding about the card fixer's runner, and possibly weaver's, getting no probe.
F5 to F8 are test-coverage gaps, and F9 is the understated Merge danger blast radius.
F2 is marked `design-wrong` because settling its premise may change the release contract, which needs a fresh review.
