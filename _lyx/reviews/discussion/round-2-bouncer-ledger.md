---
round: 2
ledger:
  - key: deadline-bound-false-for-autonomous-callers
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-fallback-contradicts-open-risks
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: hold-notice-agent-text-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: legacy-attach-and-notice-persistence-open
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d9-notice-lacks-outstanding-tasks
    rounds: [1]
    status: resolved
  - key: d8-tail-check-whitespace-unspecified
    rounds: [1]
    status: resolved
  - key: scope-omits-in-scope-decisions
    rounds: [1]
    status: resolved
  - key: decisions-without-acceptance-criteria
    rounds: [1]
    status: resolved
  - key: requirements-only-in-support-log
    rounds: [1]
    status: resolved
  - key: incident-narrative-in-d1-d3
    rounds: [1, 2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised ten findings; round 2 raised five, with a highest severity of MEDIUM.
Five round-1 keys are absent from round 2 and are marked resolved.
The other five recur in round 2, each narrower than before.

- The deadline bound (F1) fell from BLOCKING to MEDIUM and stays design-class: D4's bound still misstates the autonomous callers' deadlines, and Open risks omits the no-notifier case.
- The agent-authored hold notice text (F2) stays design-class at MEDIUM: the notice has no owner for data-not-instruction and no control-character or character-cut bound.
- The legacy asking record (F3) stays decision-class at MEDIUM: a replayed backlog may notify several old turn ends.
- The transcript-fallback wording mismatch (F4) is LOW consistency.
- The duplicated transcript citation (F5, mapped to the incident-narrative key) is a NIT relocation.
