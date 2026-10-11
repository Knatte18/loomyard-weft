---
round: 1
ledger:
  - key: seat-directives-contradict-channel-bound
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: kept-reviewer-writes-and-reply-findings-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: required-advisor-done-with-failed-gate-uncovered
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: wait-error-rule-changed-while-claimed-unchanged
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: acceptance-criteria-without-decision
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: open-risk-requirement-without-acceptance-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: keeppane-fork-audit-risk-not-real
    rounds: [1]
    status: open
    class: scope
    severity: NIT
---

Round 1 raised seven findings, all open.
The gating ones concern the discussion's seat-channel design: the shared directives conflict with the channel bound, the kept reviewer's post-acceptance writes and replies are unbounded, and a required advisor ending done with a failed gate has no path.
The rest are consistency and scope cleanups in the decision record.
