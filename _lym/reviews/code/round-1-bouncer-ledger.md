---
round: 1
ledger:
  - key: api-error-roles-under-deadline-not-opted-in
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: unfinished-omits-api-error
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: orch-role-contract-omits-api-error-cause
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: tackler-api-error-question-unasserted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pending-clear-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: usage-limit-retried-like-server-error
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: empty-api-error-text-dangling-colon
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failed-resend-logs-info-every-tick
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-completion-signal-omits-api-error
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-shuttle-bullet-states-three-outcomes
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: retry-api-error-branches-without-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: attach-and-shadow-api-error-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: duplicate-classify-deadline-expiry-call
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; no prior ledger exists.
Thirteen findings are open, none BLOCKING.
The gating ones are three MEDIUM design findings: `unfinished` treats an api-error end as finished, the tackler's api-error question is unasserted, and the clear of a pending API error is unpinned.
F1 narrows the Goal's "any other role run under a deadline" to the two settlers; it is a scope finding whose fix needs a decision, so it is `design-wrong` and goes to the parent if a role is left out on purpose.
F6 chooses between ending a limited account at once and documenting the bound, so it is also `design-wrong`.
The rest are local corrections the finding text already fixes.
