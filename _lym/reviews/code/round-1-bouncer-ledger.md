---
round: 1
ledger:
  - key: probe-error-refusal-sets-run-ended
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: bracket-verb-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: empty-failing-identities-header
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: notice-held-back-undocumented
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stuck-reason-branches-uncovered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: empty-output-tail-dangling-clause
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, none seen before.
The two MEDIUM design findings gate: a transient lock-probe error wrongly marks a refusal `run_ended`, and the bracket-verb wrapping in `addVerbs` has no test.
The four lower findings are local corrections whose fix text fixes the result.
