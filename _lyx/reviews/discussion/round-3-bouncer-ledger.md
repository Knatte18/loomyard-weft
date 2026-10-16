---
round: 3
ledger:
  - key: d1-handon-signal-matches-d2-faults
    rounds: [1]
    status: resolved
  - key: d1-settling-fork-unbounded
    rounds: [1, 2]
    status: resolved
  - key: d3-peer-forks-unlinked-to-d1-d2
    rounds: [1]
    status: resolved
  - key: d7-issue-673-move-misses-post-plan-landing
    rounds: [1, 2]
    status: resolved
  - key: d6-bound-overstates-no-override
    rounds: [1]
    status: resolved
  - key: d2-rationale-deliberation-narrative
    rounds: [1]
    status: resolved
  - key: d1-brief-step1-invites-fork-design-calls
    rounds: [2]
    status: resolved
  - key: d1-operator-label-unverified
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-board-source-no-provenance
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-support-log-not-in-brief-materials
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: role-bullet-design-call-survives-beside-d1-d2
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: decide-fragments-frame-fork-own-decision
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d9-perch-owner-fails-at-merge-budget-0
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d9-lands-first-note-no-reader
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Rounds 1 and 2 keys no longer carry a finding.
`d1-settling-fork-unbounded` closed its arbiter, parent and auto-mode routes; round 3 raises two different D1 routes under new keys (the unverified `--by operator` label, board text with no provenance).
`d7-issue-673-move-misses-post-plan-landing` has no round 3 finding; D9's owner and note gaps are filed as new keys.
`d1-brief-step1-invites-fork-design-calls` has no round 3 finding.

Round 3 raised seven findings, none BLOCKING.
The two gating MEDIUM findings are first raised in round 3, so Go carries them into Open risks.
