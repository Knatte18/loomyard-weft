---
round: 2
ledger:
  - key: merge-in-align-from-lost-on-error-stop
    rounds: [1]
    status: resolved
  - key: spent-cap-non-spent-end-no-resume
    rounds: [1]
    status: resolved
  - key: undecodable-attempt-record-halt-unmarked
    rounds: [1]
    status: resolved
  - key: d5-rubric-content-uncarried
    rounds: [1]
    status: resolved
  - key: card-51-namepr-gate-test-readers
    rounds: [1]
    status: resolved
  - key: card-13-darn-recipe-seam-list
    rounds: [1]
    status: resolved
  - key: card-26-code-review-off-nil-default
    rounds: [1]
    status: resolved
  - key: card-21-23-retired-keys-location
    rounds: [1]
    status: resolved
  - key: card-15-round-scenario-failure-rows
    rounds: [1]
    status: resolved
  - key: landingshed-doc-require-pr-paragraph
    rounds: [1]
    status: resolved
  - key: shed-recipe-spec-reject-review-name
    rounds: [1]
    status: resolved
  - key: card-11-timeout-row-no-clock
    rounds: [1]
    status: resolved
  - key: card-29-duplicate-verified-tree-row
    rounds: [1]
    status: resolved
  - key: finalize-over-cap-halt-resume-loops
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: merge-in-over-cap-note-never-cleared
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: aligned-tip-empty-on-first-merge-in
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
---

Round 1's thirteen findings are not raised again in round 2, and the review's Rejected section confirms the three focus points touching them are not defects, so all are resolved.
Round 2 raises three new findings.
F1 (BLOCKING, design): a Finalize over-cap halt at a budget-0 route back loops, because the resumed entry judges the same Baseline change by the caps again; the fix mirrors card 26's Merge-In mechanism.
F2 (MEDIUM, design): card 26's over-cap note is never cleared, so a later resume rewrites the baseline and lets earlier fixer commits escape the cap.
F3 (MEDIUM, consistency): cards 7, 11 and 26 disagree on the fixer's start tip on a first Merge-In.
All three fixes change a contract or state rule, so each is `design-wrong` and needs a fresh review.
