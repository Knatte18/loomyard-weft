---
round: 1
ledger:
  - key: pair-teardown-ends-beam-orch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: endsubject-spare-set-too-narrow
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: watcher-kills-dobby-before-archive
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: subject-end-not-shared-function
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: subject-run-selection-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pattern-child-processes-enforcement-omits-endsubject
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: failing-end-keeps-orch-strand-up
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: orch-registered-read-failure-no-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failing-second-resume-no-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: holder-read-and-carried-count-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: starter-error-test-needs-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: runproc-doc-hubowner-roles-omit-orch
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: failed-resume-row-reads-as-no-transcript
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 13 findings, none recurring.
Five MEDIUM design findings gate the loop.
They cover the orch's reach through pair teardown, the `EndSubject` spare set, the watcher's effect on the dobby's hand-over, where the subject-to-runs selection lives, and its missing test.
F1, F3, F4 and F7 change a decision or an interface, so their fixes need a fresh review.
The rest are local corrections the findings already spell out.
