---
round: 1
ledger:
  - key: card6-arbiter-attempt-restart-read
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card6-snapshot-nil-safety-untargeted
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card13-batten-drive-refusal-spec-row
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card8-driver-kind-base-layer
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card4-arbiterfake-start-resign-arrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card7-resolver-test-per-symbol
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card11-wedged-step-bound
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card13-run-tests-blocked-paused-envelopes
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review; there is no prior ledger.
The gating finding is the arbiter-attempt restart read that card 6 leaves on `loomengine` (F1), which needs a new seam and so a fresh review.
The other BLOCKING findings are local corrections: nil-safety for `TacklerDeps.Snapshot` moved into card 4 (F2) and the missing `refusal-spec.md` row for `battenDriveRefusal` (F3).
F4 closes the base-layer gap for `driver_kind` in `batten.yaml`.
F5 to F8 are LOW consistency and design corrections.
