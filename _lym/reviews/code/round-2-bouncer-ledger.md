---
round: 2
ledger:
    - key: moved-checks-way-forward-cannot-clear-halt
      rounds:
        - 1
      status: resolved
    - key: non-uncaused-ruling-reuse-across-range-change-unpinned
      rounds:
        - 1
      status: resolved
    - key: overview-omits-third-way-forward-and-rejudge
      rounds:
        - 1
      status: resolved
    - key: check-and-judge-comment-states-old-reuse-rule
      rounds:
        - 1
      status: resolved
    - key: rejudge-trigger-wording-overclaims
      rounds:
        - 1
      status: resolved
    - key: range-key-content-coverage-unpinned
      rounds:
        - 1
      status: resolved
    - key: range-key-trailer-input-redundant
      rounds:
        - 1
      status: resolved
    - key: range-key-omits-added-files
      rounds:
        - 2
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
---
Round 1 raised that the judge's file rules left the halt's way forward leading back to the halt,
along with test gaps on the reuse rule and range key, stale or overclaiming wording, and a redundant key input.
Round 2 reports all of those resolved.

Round 2 raises one new point.
The range key is built from the files that exist at the review base, so files the range adds never change it.
An uncaused ruling then stays reused when the dropped checks land in a new test file,
and a `carried_by` ruling naming an added file stays reused after that file drops the check.
The comments, `doc.go`, overview and change description claim wider coverage than the key had.
The fix changes an interface (a new drift seam over the range's full diff), so it needs a fresh review.
