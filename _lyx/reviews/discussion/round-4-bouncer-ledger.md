---
round: 4
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
    status: resolved
  - key: request-marker-outlives-watcher
    rounds: [2]
    status: resolved
  - key: skill-skip-bound-disagrees-with-orch-stencil
    rounds: [2, 3, 4]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d10-exemption-targets-go-text
    rounds: [2]
    status: resolved
  - key: d10-scan-matchable-text
    rounds: [3, 4]
    status: open
    class: decision
    severity: MEDIUM
  - key: skip-causes-disagree-with-acceptance
    rounds: [3]
    status: resolved
  - key: goal-exception-omits-orch
    rounds: [3]
    status: resolved
  - key: parent-identity-for-in-run-roles
    rounds: [3]
    status: resolved
  - key: d10-orch-exemption-stencils
    rounds: [4]
    status: open
    class: decision
    severity: MEDIUM
---

Round 4 is the first round with no design-class finding.
`d10-scan-matchable-text` persists as F1 and F4 (phrase list and exemption rule, load-a-skill match), now class `decision`.
`skill-skip-bound-disagrees-with-orch-stencil` persists as F3 (owner and delivery of the skill-load timeout), consistency MEDIUM.
New: F2 `d10-orch-exemption-stencils` (which stencils the orch exemption covers).
Round 3's `opening-stencil-per-role-undefined`, `skip-causes-disagree-with-acceptance`, `goal-exception-omits-orch` and `parent-identity-for-in-run-roles` did not reappear and are resolved.
