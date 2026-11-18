---
round: 1
ledger:
  - key: resume-startup-satisfied-by-old-events
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: resume-clears-skills-changes-session-scope
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: resume-manifest-copy-not-refreshed
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: old-strand-removed-before-record-names-new
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: step-stop-skips-attach-age-guard
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: resume-discards-unconfirmed-holder-warning
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: leftover-runs-not-reconciled-on-restep
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: resume-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: panic-orphan-hook-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-save-leaves-state-naming-removed-strand
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resume-overwrites-original-prompt
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: step-stop-reports-finished-run-as-stopped
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: summary-misstates-attach-untracked-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: orphaned-hook-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: empty-note-skip-no-row
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 filed fifteen findings, none carried from an earlier round.
The gated re-prompt's resume in `internal/shuttleengine/relaunch.go` is the main defect cluster:
startup proof read from the reused events file, skills cleared for the skip, an unrefreshed manifest copy,
a removal ahead of the record, a discarded holder warning, and thin tests.
The step stop in `stepstop.go` and the re-step reconcile in `internal/loomcli/arm.go` form the second cluster.
The `OnStepOrphaned` hook's panic and empty-note paths in `internal/shedverbs/loop.go` lack tests.
