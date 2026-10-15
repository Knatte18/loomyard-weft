---
round: 1
ledger:
  - key: d1-hint-points-at-overwritten-id
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: reed-rebirth-watcher-regresses-id
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: live-strand-rebind-unspecified-writer
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: subagent-hook-id-assumption-only-in-support-log
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: two-envelope-hints-collide
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: docs-constraint-duplicates-scope
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, none carried from an earlier round.
The core defect is that D4 overwrites the recorded session id at a fresh launch, so D1's `--continue` hint resumes the new session; the fix moves the hint to `--adopt` with the previous id read before the save.
The reed rebirth and live-strand rebind findings are further holes in who writes the id and when.
The remaining three are local text corrections.
