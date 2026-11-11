---
round: 1
ledger:
  - key: trailer-commit-not-made-when-bytes-landed
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: failed-commit-restore-refusal-loop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: decision-lock-omits-carry-over-writer
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: trailer-names-no-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-overwrites-filed-family-project-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: replace-notice-reaches-neither-directive-nor-merriam
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: live-batten-refusal-unnamed-seam
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cleared-status-fold-worktree-record-stale
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: one-write-claim-no-rollback
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: weft-history-after-board-fold-unresearched
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: carry-over-block-addition-not-refused
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: identity-text-bound-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: caller-list-omits-sampler-strand
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: note-repeats-d2
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised fourteen findings, none carried from an earlier round.
Key mapping to review IDs, in ledger order: F1 to F14.
F1 is the one BLOCKING finding: the seam D1 names makes no commit when nothing is staged, so D2's cut has no trailer.
F2 to F9 are design-class MEDIUM and gate.
F10 and F11 are design-class LOW, F12 is consistency LOW, F13 and F14 are consistency NITs; none of these gate.
