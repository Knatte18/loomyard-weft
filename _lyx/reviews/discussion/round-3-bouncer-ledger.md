---
round: 3
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
    rounds: [2, 3]
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
    status: open
    class: consistency
    severity: LOW
  - key: d6-revive-resumes-whole-pair
    rounds: [2]
    status: resolved
  - key: d7-lock-held-way-forward-names-nonexistent-status
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-no-live-driver-refusal-leads-with-noop-verb
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: halted-wait-reason-still-names-loom-start
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
---

Round 1 raised eight findings, two BLOCKING; round 2 no longer raised seven of them, and round 3 does not raise the wait-mark key either, so it is resolved.
Round 2's findings on the kill wording, the process-group package Constraint, the Goal and Scope revive overpromise, D7's bootstrap lock, D7's missing way forward, the reused refusal messages and the D6 revive scope are not raised in round 3.

Round 3 raised five findings.
The gating pair are `design` at MEDIUM and both sit in D7's refusal text.
The lock-held refusal names `lyx loom status` as showing a lock holder, which it does not (`d7-lock-held-way-forward-names-nonexistent-status`).
The no-live-driver refusal leads with `lyx batten run`, a no-op for a run with no driver strand in reed state (`d7-no-live-driver-refusal-leads-with-noop-verb`).
`halted-wait-reason-still-names-loom-start` is `consistency` at MEDIUM: batten's halted stuck reason and awaiting hint still tell the orch to run `lyx loom start`, and D6 does not name the stuck reason as the carrier.
Two LOW `consistency` findings carry over from round 2's areas: the Acceptance lock-held row is narrower than D7 (`d7-halted-lock-held-and-refusal-order`), and the verify deadline is called both a constant and a parameter with no stated supplier (`verify-deadline-no-test-seam`).
