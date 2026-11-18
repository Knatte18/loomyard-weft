---
round: 1
ledger:
  - key: holder-glyph-literal-repeated
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: parent-holds-stale-across-owner-reconcile
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: finalize-description-error-text-changed
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: parentalign-moves-scratch-before-row-entry-read
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: optional-seam-names-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: summary-comment-line-figure-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, none carried from an earlier round.
F1 (BLOCKING) is the repeated holder literal in `internal/shuttleparity/seats.go`.
F2 (MEDIUM, design) is the parent-holds read that goes stale across the owner check's reconcile in `landUnderLock`.
F3 and F4 are behaviour changes in error text and operation order that the board entry's no-behaviour-change constraint does not license.
F5 is the untested optional-seam names in the wiring check.
F6 is the obsolete comment-line figure in the change description.
