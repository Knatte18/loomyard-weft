---
round: 1
ledger:
  - key: failed-writer-ends-as-error-not-stuck
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: join-parts-unbounded-by-current-row
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: stencil-content-needs-operator-approval
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: lost-chair-discards-finished-parts
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hold-deferral-ignores-done-writer-fix
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: shared-spot-uses-unverifiable-ordering
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: part-gate-per-seat-binding-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: writer-rule-misreads-goto-recovered-run
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: archive-prefixed-part-name-escapes-archiving
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: accept-has-no-result-at-resume
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d8-acceptance-fails-for-shared-spot-exports
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: part-flag-is-sixth-mode
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings, all still open.
The gating ones are the failed-writer path (F1), the unbounded `join-parts` verb (F2), the stencil approval that only the parent can settle (F3), and the three design MEDIUMs on resume with a lost chair (F4), hold deferral for a done writer (F5) and shared-spot ordering (F6).
The remainder are LOW or NIT local corrections to the decision record.
