---
round: 1
ledger:
  - key: deadline-bound-false-for-autonomous-callers
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d2-fallback-contradicts-open-risks
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: hold-notice-agent-text-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: legacy-attach-and-notice-persistence-open
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d9-notice-lacks-outstanding-tasks
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d8-tail-check-whitespace-unspecified
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: scope-omits-in-scope-decisions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: decisions-without-acceptance-criteria
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: requirements-only-in-support-log
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: incident-narrative-in-d1-d3
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised ten findings, all still open.
The gating ones are the false 30-minute deadline bound (BLOCKING, design) and the unbounded agent-authored hold notice text (MEDIUM, design).
The rest are decision gaps, consistency mismatches, missing acceptance criteria, and two relocations in both directions: requirements to move into `decision-record.md`, and incident narrative to move out of it.
