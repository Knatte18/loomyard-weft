---
round: 2
ledger:
  - key: merge-in-and-moved-parent-check-read-different-refs
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: parent-lock-held-through-pr-rework
    rounds: [1]
    status: resolved
  - key: finalize-retry-merges-unchecked-parent
    rounds: [1]
    status: resolved
  - key: release-semantics-after-pr-gate-resume-unbounded
    rounds: [1]
    status: resolved
  - key: land-row-lock-answer-only-in-support-log
    rounds: [1]
    status: resolved
  - key: review-cut-short-by-reroute-unreachable
    rounds: [1]
    status: resolved
  - key: reroute-cap-span-and-reset-unstated
    rounds: [1]
    status: resolved
  - key: darn-no-merge-perch-stated-three-times
    rounds: [1]
    status: resolved
  - key: parent-fast-forward-moves-pair-worktree-unbounded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: push-skipped-hub-sticks-every-merge-in
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1's lock-release and retry findings did not recur in round 2, and the review's focus answers found no defect in lock release on rejection or lock record ownership after a PR-Gate resume.
The ref-model finding reopened in a sharper form: the moved-parent check reads the upstream tip, while Finalize's catch-up merge-in and parent-side merge use the local parent branch.
Two new design findings concern the fast-forward of the parent pair's worktree and a push-skipped hub.
