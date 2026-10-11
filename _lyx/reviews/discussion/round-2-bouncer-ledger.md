---
round: 2
ledger:
  - key: d7-session-end-no-launch-boundary
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: d8-post-paste-confirm-residue-open
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d8-residue-submit-amends-pattern-enter-rule
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: turn8-ledger-states-superseded-d7-fix
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: treat-as-empty-rejection-applies-to-d8
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
  - key: d7-registry-staleness-rule-unsound
    rounds: [1]
    status: resolved
  - key: d5-escaped-permission-dialog-stays-asking
    rounds: [1]
    status: resolved
  - key: d7-registry-veto-vs-fold-session-end
    rounds: [1]
    status: resolved
  - key: d1-d3-hook-order-condition-disagree
    rounds: [1]
    status: resolved
  - key: d1-blocker-leaves-task-fate-open
    rounds: [1]
    status: resolved
  - key: d8-wipe-window-unbounded
    rounds: [1]
    status: resolved
  - key: criteria-death-startup-mismatch-decisions
    rounds: [1]
    status: resolved
  - key: support-log-q2-contradicts-record
    rounds: [1]
    status: resolved
  - key: restatements-outside-decisions
    rounds: [1]
    status: resolved
---

Round 1 raised ten findings; round 2 did not re-raise nine of them, and the review's focus verdicts found D5 and D7's registry-evidence rule sound.
The one carried key is `d8-residue-submit-amends-pattern-enter-rule`, now narrowed to the background's lead sentence (merge and submit clauses) and LOW.
Round 2 adds four keys: the BLOCKING D7 `SessionEnd` launch-boundary gap, the MEDIUM D8 post-paste confirm gap, a stale Turn 8 ledger line and a NIT on the treat-as-empty rejection reason.
