---
round: 1
ledger:
  - key: merge-door-misclassified-verify-gate-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1: the change description calls the merge door two-way, though the diff drops the optional `triage` key from the persisted `verify-gate.yaml` record.
The fix is a wording change in the description; the code stays as it is.
