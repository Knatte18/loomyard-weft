---
round: 2
ledger:
  - key: join-parts-unbounded-by-current-row
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: failed-writer-ends-as-error-not-stuck
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stencil-content-needs-operator-approval
    rounds: [1]
    status: resolved
  - key: lost-chair-discards-finished-parts
    rounds: [1]
    status: resolved
  - key: hold-deferral-ignores-done-writer-fix
    rounds: [1]
    status: resolved
  - key: shared-spot-uses-unverifiable-ordering
    rounds: [1]
    status: resolved
  - key: part-gate-per-seat-binding-unstated
    rounds: [1]
    status: resolved
  - key: writer-rule-misreads-goto-recovered-run
    rounds: [1]
    status: resolved
  - key: archive-prefixed-part-name-escapes-archiving
    rounds: [1]
    status: resolved
  - key: accept-has-no-result-at-resume
    rounds: [1]
    status: resolved
  - key: d8-acceptance-fails-for-shared-spot-exports
    rounds: [1]
    status: resolved
  - key: part-flag-is-sixth-mode
    rounds: [1]
    status: resolved
  - key: resume-mid-fix-loses-fix
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: chair-timeout-equals-writer-timeout
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: parts-write-self-check-has-no-command
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-commits-on-done-only
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: open-risks-keeps-dropped-approval-line
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: next-step-ambiguous-in-chair-stencil
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parts-write-stencil-omits-cap-and-split-note
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: open-risks-line-predates-signature-allowance
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings.
Round 2 reopened two of them: the `join-parts` bound (BLOCKING, still false for halted runs, a pre-resume goto and live part writers) and the failed-writer mapping (MEDIUM, a writer timeout is unmapped and the died or not-started error is misstated).
The stencil-approval decision finding is settled by the parent's `Added after Discussion` entry.
The other round 1 findings were not raised again and are resolved; round 2 states that the hold-deferral and live-chair resume rules hold.

Round 2 added new keys: a resume mid-fix that loses the fix, a chair timeout equal to a writer's, a Parts-Write self-check without a command, and five local consistency or stencil corrections.
