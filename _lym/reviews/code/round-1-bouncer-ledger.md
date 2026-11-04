---
round: 1
ledger:
  - key: parent-side-merge-ignores-forced-squash
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: reopen-atonce-flag-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pr-record-fields-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: closed-record-halt-names-noop-pr-review
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: protected-branch-advice-tautological-under-merge
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prlanding-merge-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings: F1 and F2 at MEDIUM and gating, F3 to F5 at LOW, F6 a NIT.
F1 is a correctness gap in `landUnderLock`: under a merge parent the parent-side merge takes `Squash` from config while GitHub always squashes.
F2 and F3 are coverage gaps in `TestFinalize_PullRequest`.
F4 is a halt that names a step that changes nothing under merge; its fix chooses between a behavior change and a wording change, so it is marked `design-wrong`.
F5 is advice that is a no-op when the parent is already merge.
F6 is a stale field comment.
No ledger existed before this round.
