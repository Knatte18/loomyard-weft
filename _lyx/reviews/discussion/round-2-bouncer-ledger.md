---
round: 2
ledger:
  - key: decision-9-multi-package-rerun-and-splice
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
  - key: reason-location-and-move-then-delete-record
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: keep-criterion-checkability-and-rerun-stop
    rounds: [1]
    status: resolved
  - key: core-testkit-edit-collision
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: coverage-method-coverpkg-and-drop-rule
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: unused-helper-tag-set
    rounds: [1]
    status: resolved
  - key: unmade-tier-move-reason-home
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: receiver-list-acceptance-check
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: isolated-rerun-command-package
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 opened six findings; round 2 raised seven.
Two round-1 keys are resolved: the keep-criterion checkability and rerun stop, and the unused-helper tag set.
Four round-1 keys reappear in round 2 in new form.

- `core-testkit-edit-collision` (F1) is now BLOCKING: a candidate kept because a core or testkit file names it has no reason class that Decision 10 allows.
- `coverage-method-coverpkg-and-drop-rule` (F2) is MEDIUM: the drop rule contradicts its rationale, and the per-package figure under `-coverpkg=./...` is undefined.
- `reason-location-and-move-then-delete-record` (F3) is design MEDIUM: a test that received a moved assertion can be deleted or left without a fitting keep reason on a later rerun.
- `decision-9-multi-package-rerun-and-splice` (F4) is MEDIUM: the splice has no owner, and Decision 9 and Decision 10 disagree on how often the rerun runs.

New in round 2: F5 (`unmade-tier-move-reason-home`, design MEDIUM), F6 (`receiver-list-acceptance-check`, LOW) and F7 (`isolated-rerun-command-package`, NIT).
Gating by class and severity: F1, F3 and F5.
