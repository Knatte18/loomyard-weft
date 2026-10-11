---
round: 2
ledger:
  - key: seat-directives-contradict-channel-bound
    rounds: [1]
    status: resolved
  - key: kept-reviewer-writes-and-reply-findings-unbounded
    rounds: [1]
    status: resolved
  - key: required-advisor-done-with-failed-gate-uncovered
    rounds: [1]
    status: resolved
  - key: wait-error-rule-changed-while-claimed-unchanged
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-without-decision
    rounds: [1]
    status: resolved
  - key: open-risk-requirement-without-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: keeppane-fork-audit-risk-not-real
    rounds: [1]
    status: resolved
  - key: required-advisor-not-stopped-on-early-end
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: chair-notice-deferral-bound-vacuous
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 1's seven findings are resolved: round 2's review raised none of them again and its focus verdicts found the seat-directive and kept-reviewer questions not to be defects.
Round 2 raised two new findings.
The early end of a step does not stop the required advisor that triggered it, and D10's deferral bound cannot fire before the chair's own deadline.
