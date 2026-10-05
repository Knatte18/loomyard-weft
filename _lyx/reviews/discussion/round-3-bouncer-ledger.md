---
round: 3
ledger:
  - key: operator-ban-contradicts-interactive-discussion
    rounds: [1]
    status: resolved
  - key: restart-never-skips-vs-timeout-skip
    rounds: [1]
    status: resolved
  - key: compact-cycle-unbounded
    rounds: [1]
    status: resolved
  - key: skipped-skill-unbounded
    rounds: [1]
    status: resolved
  - key: directive-role-set-undefined
    rounds: [1]
    status: resolved
  - key: parent-message-recognition
    rounds: [1]
    status: resolved
  - key: rationale-narrative
    rounds: [1]
    status: resolved
  - key: opening-stencil-per-role-undefined
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: request-marker-outlives-watcher
    rounds: [2]
    status: resolved
  - key: skill-skip-bound-disagrees-with-orch-stencil
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d10-exemption-targets-go-text
    rounds: [2]
    status: resolved
  - key: d10-scan-matchable-text
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: skip-causes-disagree-with-acceptance
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: goal-exception-omits-orch
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: parent-identity-for-in-run-roles
    rounds: [3]
    status: open
    class: decision
    severity: LOW
---

Round 1's findings stayed resolved.
Round 2's `request-marker-outlives-watcher` and `d10-exemption-targets-go-text` did not reappear and are resolved.
`opening-stencil-per-role-undefined` shrank to one omission (the burler row), now LOW (F6).
`skill-skip-bound-disagrees-with-orch-stencil` persists as F1 (the skip bound has two owners), MEDIUM consistency.
New in round 3: F3 `d10-scan-matchable-text` (design MEDIUM, the gating one), F2 `skip-causes-disagree-with-acceptance`, F4 `goal-exception-omits-orch`, F5 `parent-identity-for-in-run-roles`.
