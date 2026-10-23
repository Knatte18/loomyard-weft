---
round: 3
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
    status: resolved
  - key: merge-in-over-cap-note-never-cleared
    rounds: [2]
    status: resolved
  - key: aligned-tip-empty-on-first-merge-in
    rounds: [2]
    status: resolved
  - key: card-29-over-cap-resume-failing-gate-untested
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: cards-35-36-per-kind-landing-halt-rows
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card-40-kept-lock-bound-unstated
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card-7-merge-in-empty-aligned-tip-optional
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 raised sixteen keys; none is raised in round 3, so all are resolved.
Round 3 raises four new keys, none BLOCKING: a test-row gap for the over-cap resume over a failing gate at Finalize (F1), per-kind landing-halt rows repeating the predicate test (F2), an unstated bound on the lock card 40 keeps (F3), and an `AlignedTip` optionality that should be finalize-mode only (F4).
Each fix is text the finding already specifies, so all are `mechanical`.
