---
round: 2
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
    status: open
    class: design
    severity: MEDIUM
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
    status: open
    class: consistency
    severity: MEDIUM
  - key: process-group-constraint-unimportable-package
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: verify-deadline-no-test-seam
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: goal-scope-never-revives-overpromise
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d7-bootstrap-lock-unspecified
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d7-refusals-no-way-forward
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d7-reused-refusal-messages-name-start
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d7-halted-lock-held-and-refusal-order
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: d6-revive-resumes-whole-pair
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 1 raised eight findings, two BLOCKING (the Windows kill premise and the gated-strand drop).
Round 2's fresh review no longer raises seven of them, so those keys are resolved.
The wait-mark key (F5) is the one that carries over: the failure bound is still unstated for the mark op's own errors, at shuttle's sites and in the landing callback.

Round 2 raised nine new keys.
The gating ones are the wait-mark key and the D6 revive scope (F10), both `design` at MEDIUM.
The kill wording (F1), the process-group package Constraint (F2), the Goal and Scope revive overpromise (F4) are `consistency` at MEDIUM; D7's bootstrap lock (F6) and way-forward gaps (F7) are `decision` at MEDIUM.
The rest are LOW: the missing deadline test seam (F3), reused refusal messages (F8) and the halted-with-lock-held case (F9).
