---
round: 1
ledger:
  - key: pattern-md-unrelated-entries-shortened
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: runproc-end-group-members-not-watched
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: runproc-failed-signal-not-reported
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pattern-overview-declared-entries-unchanged
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
---

Round 1 raised four findings.
The gating one is the `runproc.End` group handling: only the recorded leader is watched, so a surviving group member is neither escalated nor reported.
The others are the `PATTERN.md` entries shortened without a declared change, a failed signal that goes unreported, and the overview lines of two declared entries that stay unchanged.
