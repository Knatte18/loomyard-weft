---
round: 3
ledger:
  - key: d7-session-end-no-launch-boundary
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-residue-submit-amends-pattern-enter-rule
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: NIT
  - key: d8-post-clear-reading-guard-unbounded
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d8-post-paste-confirm-residue-open
    rounds: [2]
    status: resolved
  - key: turn8-ledger-states-superseded-d7-fix
    rounds: [2]
    status: resolved
  - key: treat-as-empty-rejection-applies-to-d8
    rounds: [2]
    status: resolved
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

Round 1 raised ten findings, round 2 five and round 3 three; none of round 3 is BLOCKING.
The D7 `SessionEnd` launch-boundary key narrowed from BLOCKING to MEDIUM: the offset rule closed the not-yet-registered half, and the read-alive disjunct it added leaves the resumed-launch window open.
The D8 confirm, Turn 8 ledger and treat-as-empty keys were found sound in round 3's focus verdicts and are resolved.
The PATTERN-line key is down to a NIT about the Constraints paraphrase.
Round 3 adds one key: D8's post-clear reading is not stated to be a full guard reading, and the guard's one-fresh-reading clause is not among the amended places.
