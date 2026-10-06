---
round: 5
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
    status: resolved
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
    status: resolved
  - key: isolated-rerun-command-package
    rounds: [2]
    status: resolved
  - key: removable-no-candidate-no-outcome
    rounds: [3]
    status: resolved
  - key: coverage-figure-directory-filter
    rounds: [3, 4]
    status: resolved
  - key: decision-9-no-tests-clause
    rounds: [3]
    status: resolved
  - key: tier-move-comment-spelling
    rounds: [3]
    status: resolved
  - key: scenario-merge-per-family-measurement
    rounds: [3, 4, 5]
    status: open
    class: decision
    severity: MEDIUM
  - key: scenario-steps-isolation-bound
    rounds: [3, 4, 5]
    status: open
    class: design
    severity: MEDIUM
  - key: scenario-decision-form-and-coverage
    rounds: [3, 4, 5]
    status: open
    class: consistency
    severity: LOW
  - key: scenario-parallel-stop-on-failure
    rounds: [4]
    status: resolved
  - key: scenario-alone-rerun-mapping
    rounds: [4, 5]
    status: open
    class: consistency
    severity: LOW
  - key: support-log-leftovers
    rounds: [4]
    status: resolved
  - key: scenario-merge-tier-bound
    rounds: [5]
    status: open
    class: design
    severity: MEDIUM
  - key: scenario-serial-vs-fold-parallel
    rounds: [5]
    status: open
    class: consistency
    severity: LOW
---

Rounds 1 to 4 opened six, seven, nine and seven findings; round 5 opened six.

Resolved and not raised in round 5: `coverage-figure-directory-filter`, `scenario-parallel-stop-on-failure` and `support-log-leftovers`, plus every earlier key.

Open in round 5:

- `scenario-steps-isolation-bound` (F2, design MEDIUM, gating, recurring since round 3): the restore clause defines no observable state and no check.
  Earlier rounds filed it as decision; F2's own entry argues the design is missing a bound, and the class is taken from F2.
- `scenario-merge-tier-bound` (F1, design MEDIUM, gating, new): a merge can raise a test's tier out of the gates that ran it.
- `scenario-merge-per-family-measurement` (F3, decision MEDIUM, recurring since round 3): spread, measured time, load tolerance and run order are open.
- `scenario-alone-rerun-mapping` (F4, consistency LOW, since round 4): Decision 8's rerun has no `-count=1` and a zero-match run is not ruled out as a verdict.
- `scenario-serial-vs-fold-parallel` (F5, consistency LOW, new): Decisions 4, 13 and 2 do not say which governs a scenario or whether a protected test may host one.
- `scenario-decision-form-and-coverage` (F6, consistency LOW, since round 3): Decision 10 and the Goal omit the merge.
