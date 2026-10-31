---
round: 1
ledger:
  - key: decision-add-beam-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: with-project-directive-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: record-choice-wiring-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: goto-preflight-lifts-body-lock
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: prerun-seed-read-no-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: directive-test-asserts-static-prose
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: beam-no-orch-misleading-refusal
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: repeated-path-assertions
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 (first round, no prior ledger) raised eight findings, none BLOCKING.
Two MEDIUM `scope` findings are test coverage gaps for `decision add` on a beam and for `withProjectDirective`.
The one `design` finding is LOW: a goto to Preflight lifts the body lock over an existing record.
The rest are LOW and NIT consistency or scope items with local fixes.
