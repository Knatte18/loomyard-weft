---
round: 4
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
    status: open
    class: decision
    severity: LOW
  - key: decision-9-no-tests-clause
    rounds: [3]
    status: resolved
  - key: tier-move-comment-spelling
    rounds: [3]
    status: resolved
  - key: scenario-merge-per-family-measurement
    rounds: [3, 4]
    status: open
    class: decision
    severity: MEDIUM
  - key: scenario-steps-isolation-bound
    rounds: [3, 4]
    status: open
    class: decision
    severity: MEDIUM
  - key: scenario-decision-form-and-coverage
    rounds: [3, 4]
    status: open
    class: consistency
    severity: LOW
  - key: scenario-parallel-stop-on-failure
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
  - key: scenario-alone-rerun-mapping
    rounds: [4]
    status: open
    class: decision
    severity: LOW
  - key: support-log-leftovers
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
---

Round 1 opened six findings, round 2 seven, round 3 nine, round 4 seven.

Resolved and not raised again in round 4: the core-testkit edit collision, the receiver-list acceptance check, the `Removable: no` outcome, the Decision 9 no-tests clause and the tier-move comment spelling, plus the earlier round-1 and round-2 keys.

Still open in round 4:

- `scenario-steps-isolation-bound` (F1, decision MEDIUM): the or-clause lets any step merge once a comment names its mutation.
- `scenario-merge-per-family-measurement` (F3, decision MEDIUM): no margin, tie rule, load record, `-count=1` or named package.
- `coverage-figure-directory-filter` (F5, decision LOW): the directory match is now exact; statement counting and the noise criterion remain.
- `scenario-decision-form-and-coverage` (F6, consistency LOW): Decisions 2 and 12 omit merging.

New in round 4:

- `scenario-parallel-stop-on-failure` (F2, design MEDIUM, the one gating finding): `t.Run` returns true before a parallel step runs, so the stop never fires.
- `scenario-alone-rerun-mapping` (F4, decision LOW): Decision 8's `-run` rerun passes vacuously on a merged name.
- `support-log-leftovers` (F7, consistency NIT): the placeholder line and a provenance sentence in the wrong file.
