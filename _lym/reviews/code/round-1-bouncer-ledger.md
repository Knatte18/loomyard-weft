---
round: 1
ledger:
  - key: mutation-step-go-literal-branch
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: failed-check-report-matches-head
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: gateslot-doc-omits-mutation-step
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: mutation-step-judge-nothing-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: mutation-step-findings-lists-unasserted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: report-matches-no-false-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: logsonly-side-effect-condition
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: noise-rules-missing-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: note-branches-missing-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: summary-overstates-spread
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: non-go-inputs-escape-match
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round and has no earlier ledger.
F1 is the gating blocker: the new `go` scoper literal branch in `mutationStep.run` needs a row-keyed capability lookup.
F2, F4 and F5 are gating MEDIUM design findings about a failed check matching HEAD and two untested paths of the step.
F3 and F10 are consistency findings, and F6 through F9 and F11 are LOW test-coverage and bound-statement findings.
