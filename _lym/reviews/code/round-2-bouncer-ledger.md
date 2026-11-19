---
round: 2
ledger:
  - key: mutation-step-go-literal-branch
    rounds: [1]
    status: resolved
  - key: failed-check-report-matches-head
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gateslot-doc-omits-mutation-step
    rounds: [1]
    status: resolved
  - key: mutation-step-judge-nothing-untested
    rounds: [1]
    status: resolved
  - key: mutation-step-findings-lists-unasserted
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: report-matches-no-false-row
    rounds: [1]
    status: resolved
  - key: logsonly-side-effect-condition
    rounds: [1]
    status: resolved
  - key: noise-rules-missing-rows
    rounds: [1]
    status: resolved
  - key: note-branches-missing-rows
    rounds: [1]
    status: resolved
  - key: summary-overstates-spread
    rounds: [1]
    status: resolved
  - key: non-go-inputs-escape-match
    rounds: [1]
    status: resolved
---

Round 1 raised eleven findings, one BLOCKING.
Round 2 resolved nine of them, the BLOCKING go-literal branch among them.
Two keys stay open at LOW, each with half its round-1 finding still unfixed.
`failed-check-report-matches-head` keeps the partial findings a failed check should return; the stale-match half is fixed.
`mutation-step-findings-lists-unasserted` keeps the unjudged and not-judged lists without an assertion in `TestMutationStep_RecordFindings`.
