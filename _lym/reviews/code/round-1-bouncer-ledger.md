---
round: 1
ledger:
  - key: empty-target-id-guard-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: missing-target-snapshot-record-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: target-snapshot-paths-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: blast-radius-omits-review-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all new.
The gating one is the missing test for the `record.TargetID == ""` guard in `roundUnchanged` and `fixLeftTargetUnchanged`.
The others are a missing test for a failed `Take`, a stale doc comment on `TargetSnapshot.Paths`
and a blast radius in the change description that leaves out the review row.
The review rejected one scope candidate, that "fail loudly" was only partly built because callers log a warning and continue.
