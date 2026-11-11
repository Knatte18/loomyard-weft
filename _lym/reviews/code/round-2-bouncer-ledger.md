---
round: 2
ledger:
  - key: probe-error-refusal-sets-run-ended
    rounds: [1]
    status: resolved
  - key: bracket-verb-wiring-untested
    rounds: [1]
    status: resolved
  - key: empty-failing-identities-header
    rounds: [1]
    status: resolved
  - key: notice-held-back-undocumented
    rounds: [1]
    status: resolved
  - key: stuck-reason-branches-uncovered
    rounds: [1]
    status: resolved
  - key: empty-output-tail-dangling-clause
    rounds: [1]
    status: resolved
  - key: findings-text-attempt-over-limit
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: master-rung-missing-for-probe-failure-refusal
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

All six round-1 findings were checked in round 2 and rejected, so they are resolved.
Round 2 raised two new consistency findings: the rendered findings text can still read "attempt 4 of 3", and the Master stencil has no rung for the probe-failure refusal.
Both have a local fix their text already settles.
