---
round: 4
ledger:
  - key: windows-tree-kill-false-premise
    rounds: [1]
    status: resolved
  - key: gated-strand-dropped-without-bound
    rounds: [1]
    status: resolved
  - key: batten-run-outcomes-unordered
    rounds: [1]
    status: resolved
  - key: pattern-entry-missing-for-new-invariants
    rounds: [1]
    status: resolved
  - key: wait-mark-stale-or-misdirected-unbounded
    rounds: [1, 2]
    status: resolved
  - key: revive-and-way-forward-disagree
    rounds: [1]
    status: resolved
  - key: goal-acceptance-overreach-scope
    rounds: [1]
    status: resolved
  - key: history-provenance-in-decision-record
    rounds: [1]
    status: resolved
  - key: tree-kill-wording-overpromise
    rounds: [2]
    status: resolved
  - key: process-group-constraint-unimportable-package
    rounds: [2]
    status: resolved
  - key: verify-deadline-no-test-seam
    rounds: [2, 3, 4]
    status: open
    class: consistency
    severity: LOW
  - key: goal-scope-never-revives-overpromise
    rounds: [2]
    status: resolved
  - key: d7-bootstrap-lock-unspecified
    rounds: [2]
    status: resolved
  - key: d7-refusals-no-way-forward
    rounds: [2]
    status: resolved
  - key: d7-reused-refusal-messages-name-start
    rounds: [2]
    status: resolved
  - key: d7-halted-lock-held-and-refusal-order
    rounds: [2, 3]
    status: resolved
  - key: d6-revive-resumes-whole-pair
    rounds: [2]
    status: resolved
  - key: d7-lock-held-way-forward-names-nonexistent-status
    rounds: [3]
    status: resolved
  - key: d7-no-live-driver-refusal-leads-with-noop-verb
    rounds: [3, 4]
    status: open
    class: decision
    severity: LOW
  - key: halted-wait-reason-still-names-loom-start
    rounds: [3, 4]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d6-way-forward-venue-not-shown-by-batten-status
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-d-acceptance-no-driver-strand-wording
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: d7-retiring-driver-strand-unclassified
    rounds: [4]
    status: open
    class: decision
    severity: LOW
  - key: d8-omits-overview-doc
    rounds: [4]
    status: open
    class: scope
    severity: LOW
---

Rounds 1 to 3 resolved the kill wording, the wait mark, the revive scope, the D7 bootstrap lock, the D7 lock-held and refusal-order items, and the lock-held status premise.

Round 4 raised seven findings, none BLOCKING.
The one gating finding is `d6-way-forward-venue-not-shown-by-batten-status` (F1, `design`, MEDIUM): D6 names `lyx batten status` as showing the halted child's way forward, but an exempt halt never reaches the batten state that adds `stuck_reason`.
`halted-wait-reason-still-names-loom-start` (F2) continues round 3's key as MEDIUM `consistency`: the awaiting child's hint must stay on `lyx loom start`, and only the halted reason moves to `lyx loom resume`.
`d7-no-live-driver-refusal-leads-with-noop-verb` continues as F4, a LOW `decision` finding: the dead-strand refusal should name both `lyx batten run` and `lyx loom start --no-attach` with their conditions.
`verify-deadline-no-test-seam` continues as F6, LOW: use one term and name `verifytree` as the constant's declarer.
New LOW items are F3 (no-driver-strand wording), F5 (retiring driver strand) and F7 (D8 omits `docs/overview.md`).
