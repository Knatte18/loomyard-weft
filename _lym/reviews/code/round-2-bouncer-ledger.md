---
round: 2
ledger:
  - key: notice-memory-restart-untested
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: run-selection-untested
    rounds: [1]
    status: resolved
  - key: respawn-cap-stale-and-memory-error-hides-run
    rounds: [1]
    status: resolved
  - key: respawn-cap-declared-twice
    rounds: [1]
    status: resolved
  - key: zero-quiet-min-disables-notice
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: dead-reset-helpers-battenshed
    rounds: [1]
    status: resolved
  - key: zero-quiet-window-untested
    rounds: [1]
    status: resolved
  - key: readnoticechild-lock-fallback-untested
    rounds: [1]
    status: resolved
  - key: delivery-attempt-cleanup-untested
    rounds: [1]
    status: resolved
---

Round 1 raised nine findings; round 2 raised two LOW design findings.
Run selection and the `readNoticeChild` fallback are settled by the round 2 verdicts on the focus questions.
The restart gap narrowed to the episode `Since` and the revive streak read-back (F1).
The zero `notice_quiet_min` behaviour is unchanged; what remains is the untested count-to-duration fallback in `runShedDuration` (F2).
The other round 1 keys no longer appear and are resolved.
