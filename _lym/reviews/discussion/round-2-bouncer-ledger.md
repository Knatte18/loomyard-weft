---
round: 2
ledger:
  - key: trailer-commit-not-made-when-bytes-landed
    rounds: [1]
    status: resolved
  - key: failed-commit-restore-refusal-loop
    rounds: [1]
    status: resolved
  - key: decision-lock-omits-carry-over-writer
    rounds: [1]
    status: resolved
  - key: trailer-names-no-record
    rounds: [1]
    status: resolved
  - key: d3-overwrites-filed-family-project-record
    rounds: [1]
    status: resolved
  - key: replace-notice-reaches-neither-directive-nor-merriam
    rounds: [1]
    status: resolved
  - key: live-batten-refusal-unnamed-seam
    rounds: [1]
    status: resolved
  - key: cleared-status-fold-worktree-record-stale
    rounds: [1]
    status: resolved
  - key: one-write-claim-no-rollback
    rounds: [1]
    status: resolved
  - key: weft-history-after-board-fold-unresearched
    rounds: [1]
    status: resolved
  - key: carry-over-block-addition-not-refused
    rounds: [1]
    status: resolved
  - key: identity-text-bound-overstated
    rounds: [1]
    status: resolved
  - key: caller-list-omits-sampler-strand
    rounds: [1]
    status: resolved
  - key: note-repeats-d2
    rounds: [1]
    status: resolved
  - key: d2-history-read-loom-shaped-in-fabric
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: adopted-live-motif-weft-keeps-stale-record
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-copy-fires-in-reentered-discussion
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fold-before-webster-hides-added-entry-from-master
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-cut-moves-past-later-committed-drop
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-fold-drops-record-write-precedence
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-filed-refusal-misses-fileraddle-crash-window
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d1-bound-misdescribes-unreviewed-changes
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d2-d4-bounds-overstate-guards
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: fold-makes-run-report-undercount
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: refusal-way-forward-constraint-names-wrong-decisions
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: note-defers-empty-commit-fact
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: support-log-still-says-d4-checks-batten
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: deliberation-sits-in-decision-record
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d5-stamp-unbounded-under-mixed-binaries
    rounds: [2]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1's fourteen keys are all resolved: none of round 2's fifteen findings restates one.
Round 2 mapping to review IDs, in ledger order from `d2-history-read-loom-shaped-in-fabric`: F1 to F15, with F13 as `support-log-still-says-d4-checks-batten`, F14 as `deliberation-sits-in-decision-record` and F12 as `note-defers-empty-commit-fact`.
F1 is BLOCKING: D2 puts a loom-shaped trailer parse and history cut in `internal/fabricengine`.
F2 to F6 are design-class MEDIUM and gating once they recur; all are first raised this round.
F7 to F15 are LOW or NIT and do not gate.
