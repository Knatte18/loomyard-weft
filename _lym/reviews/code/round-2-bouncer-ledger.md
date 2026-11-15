---
round: 2
ledger:
  - key: pair-teardown-ends-beam-orch
    rounds: [1]
    status: resolved
  - key: endsubject-spare-set-too-narrow
    rounds: [1]
    status: resolved
  - key: watcher-kills-dobby-before-archive
    rounds: [1]
    status: resolved
  - key: subject-end-not-shared-function
    rounds: [1]
    status: resolved
  - key: subject-run-selection-untested
    rounds: [1]
    status: resolved
  - key: pattern-child-processes-enforcement-omits-endsubject
    rounds: [1]
    status: resolved
  - key: failing-end-keeps-orch-strand-up
    rounds: [1]
    status: resolved
  - key: orch-registered-read-failure-no-row
    rounds: [1]
    status: resolved
  - key: failing-second-resume-no-row
    rounds: [1]
    status: resolved
  - key: holder-read-and-carried-count-untested
    rounds: [1]
    status: resolved
  - key: starter-error-test-needs-keep-marker
    rounds: [1]
    status: resolved
  - key: runproc-doc-hubowner-roles-omit-orch
    rounds: [1]
    status: resolved
  - key: failed-resume-row-reads-as-no-transcript
    rounds: [1]
    status: resolved
  - key: summary-describes-round-1-code
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: failed-end-hides-owner-exit-reason
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pairteardown-doc-omits-endsubjectprocesses
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's thirteen findings are all absent from round 2's review, so each is resolved.
Round 2 raised three new findings.
F1 is a stale change description, F2 a lost watcher exit reason after a failed end, and F3 a missing package doc section.
None is BLOCKING, and the one design finding is LOW, so the loop converges.
