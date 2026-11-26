---
round: 3
focus:
  - "internal/landingshed/prreview.go, openPullRequest (commits f3c8b0187, ee46a44be, 1966e5525): at the row's entry, does every found pull request whose review has not ended have GitHub's head judged before the task branch is pushed, so that a hand commit on the GitHub branch ends in the foreign-head halt and never in a push halt? Settle it for a task head equal to the gated head and for one that differs, on a resume after a closed-pull-request, dirty-tree or foreign-head halt, and on a driver restart."
  - "Same function: with the status read moved relative to the entry push, are the other paths still correct: no pull request yet (the push must precede the open), a closed or merged pull request, an ended review, a failing status read, and a task head the gate has not passed (it must not be pushed at entry)?"
  - "internal/landingshed tests, TestPRReview_Wait and TestPRReview_Call: is there a row whose push seam fails as a non-fast-forward over a hand commit, and would it fail if the entry push ran ahead of the status read? Does the wait world's always-succeeding push fake hide any other ordering?"
  - "The halt texts, doc comments and docs/overview.md around PR-Review's entry and its hand-commit halt: do they match the code's order, including which pull request states the entry check applies to and what each sentence's 'that ending' refers to?"
---

The focus is PR-Review's order of push and GitHub head read at the row's entry.
