---
round: 1
ledger:
  - key: idle-wake-in-waiter-not-daemon
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: stale-verb-comments-name-claude-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: waiter-codex-binary-not-pinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: leftover-claudeengine-indirections
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1, no prior ledger.
F1 (`idle-wake-in-waiter-not-daemon`) is an unrecorded deviation from the Goal; the code stays and the parent records or refuses it.
F2, F3 and F4 are local corrections whose fix text names the exact edit.
Nothing gates, so the loop converges at round 1.
