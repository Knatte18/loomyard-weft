---
round: 2
ledger:
  - key: reach-note-misses-writer
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: mutation-trial-order-parent-call
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: triage-lines-cut-failure-report
    rounds: [1]
    status: resolved
  - key: refresh-ignores-linked-package-dirt
    rounds: [1]
    status: resolved
  - key: new-function-cross-package-false-unreached
    rounds: [1]
    status: resolved
  - key: triage-no-change-set-false-signal
    rounds: [1]
    status: resolved
  - key: prime-mutation-judges-nothing
    rounds: [1]
    status: resolved
  - key: index-stamp-misses-added-map
    rounds: [1]
    status: resolved
  - key: function-identity-includes-file
    rounds: [1]
    status: resolved
  - key: red-tier-reach-report-matches
    rounds: [1]
    status: resolved
  - key: d6-since-branch-with-function-form
    rounds: [1]
    status: resolved
  - key: d8-hands-d2-two-commits
    rounds: [1]
    status: resolved
  - key: batch-risk-overstates-dirty-case
    rounds: [1]
    status: resolved
  - key: d8-command-cut-by-line-bytes
    rounds: [1]
    status: resolved
  - key: auto-mode-bullet-repeats-d11
    rounds: [1]
    status: resolved
---

Round 1 raised fifteen findings; round 2's review marks all but F1 resolved.
F1 returned in a new form: the reach turn's failing result goes to Darn-Write's cold fixer, which re-runs the gate and passes it, so the writer never sees the note.
F6 of round 1, the mutation order, was settled by the parent's added entry; round 2 finds that entry's six writer edits unapplied.
Round 2 also kept eight lower-severity new items out under the incremental bar.
