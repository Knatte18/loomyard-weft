---
round: 5
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
    rounds: [2, 3, 4, 5]
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
    status: resolved
  - key: halted-wait-reason-still-names-loom-start
    rounds: [3, 4]
    status: resolved
  - key: d6-way-forward-venue-not-shown-by-batten-status
    rounds: [4]
    status: resolved
  - key: d6-d-acceptance-no-driver-strand-wording
    rounds: [4]
    status: resolved
  - key: d7-retiring-driver-strand-unclassified
    rounds: [4]
    status: resolved
  - key: d8-omits-overview-doc
    rounds: [4]
    status: resolved
  - key: d7-retiring-strand-refusal-names-skipping-revive
    rounds: [5]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-awaiting-refusal-vs-hint-verbs
    rounds: [5]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d7-rationale-misdescribes-loom-start
    rounds: [5]
    status: open
    class: design
    severity: LOW
  - key: record-names-hidden-no-attach-flag
    rounds: [5]
    status: open
    class: consistency
    severity: LOW
  - key: record-run-lock-names-two-locks
    rounds: [5]
    status: open
    class: consistency
    severity: LOW
---

Rounds 1 to 4 resolved the kill wording, the wait mark, the revive scope, the D7 bootstrap and lock-held items, the halted reason, the D6 venue, the no-driver-strand wording, the retiring-strand classification and the D8 docs scope.
Round 5 confirmed the venue and the D8 docs scope as correct, and no longer raises the halted reason, the dead-strand refusal's verb order or the Acceptance wording.

Round 5 raised six findings, none BLOCKING.
The one gating finding is `d7-retiring-strand-refusal-names-skipping-revive` (F1, `design`, MEDIUM): D7's dead-strand refusal names `lyx batten run`, but D6's revive skips a live strand and reed reports a retiring one as live, so the verb cannot revive it; a retiring strand's refusal should name `lyx loom start`.
`d7-awaiting-refusal-vs-hint-verbs` (F2, `consistency`, MEDIUM) asks the record to say which verb fits which awaiting sub-state.
`verify-deadline-no-test-seam` continues as F5, LOW: the D3 term "deadline" for a duration, and the unnamed constant.
New LOW items are F3 (D7's rationale misdescribes `lyx loom start`), F4 (the hidden `--no-attach` flag) and F6 ("the run lock" names two locks).
