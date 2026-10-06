---
round: 3
ledger:
  - key: decision-9-multi-package-rerun-and-splice
    rounds: [1, 2]
    status: resolved
  - key: reason-location-and-move-then-delete-record
    rounds: [1, 2]
    status: resolved
  - key: keep-criterion-checkability-and-rerun-stop
    rounds: [1]
    status: resolved
  - key: core-testkit-edit-collision
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: coverage-method-coverpkg-and-drop-rule
    rounds: [1, 2]
    status: resolved
  - key: unused-helper-tag-set
    rounds: [1]
    status: resolved
  - key: unmade-tier-move-reason-home
    rounds: [2]
    status: resolved
  - key: receiver-list-acceptance-check
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: isolated-rerun-command-package
    rounds: [2]
    status: resolved
  - key: removable-no-candidate-no-outcome
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
  - key: coverage-figure-directory-filter
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: decision-9-no-tests-clause
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: tier-move-comment-spelling
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: scenario-merge-per-family-measurement
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: scenario-steps-isolation-bound
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: scenario-decision-form-and-coverage
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 opened six findings, round 2 seven, round 3 nine.

Resolved by round 3, not raised again: the splice and rerun frequency of Decision 9, the move-then-delete record, the coverpkg and drop rule, the tier-move reason home and the isolated-rerun command package, along with the two round-1 keys closed earlier.

Still open or new in round 3:

- `core-testkit-edit-collision` (F2) reappears as a fold that can rename or absorb a test the core or testkit files name, or a receiving test.
- `receiver-list-acceptance-check` (F3) rises from LOW to MEDIUM: no acceptance criterion can fail for an incomplete receiver list.
- `removable-no-candidate-no-outcome` (F1): a `Removable: no` candidate with no unique assertion and no fold has no permitted outcome.
- `coverage-figure-directory-filter` (F4, design): exact or prefix directory match for nested in-scope packages, statement counting and profile mode.
- `decision-9-no-tests-clause` (F5) and `tier-move-comment-spelling` (F6) are LOW consistency.
- `scenario-merge-per-family-measurement` (F7) and `scenario-steps-isolation-bound` (F8) are design MEDIUM, from the scenario decision added after Discussion.
- `scenario-decision-form-and-coverage` (F9): the scenario decision is unnumbered and Decisions 1, 2 and 7 and the acceptance criteria do not name merges.

Gating: F4, F7 and F8.
