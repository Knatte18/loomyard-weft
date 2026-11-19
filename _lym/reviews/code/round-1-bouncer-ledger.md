---
round: 1
ledger:
  - key: role-intake-split-note-duplicates-across-forks
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: list-full-comment-error-omits-issue-number
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: comment-list-failure-row-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 (split-note duplication across overlapping intake forks), F2 (comment-fetch error without the issue number) and F3 (CommentListFailure row asserting no error text).
All three fixes are local corrections whose text the review already fixes.
No finding is gating, so the loop converges.
