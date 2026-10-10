---
round: 2
ledger:
  - key: d11-skip-rule-keys-on-hook-event
    rounds: [1]
    status: resolved
  - key: d11-prompt-choice-not-fixed-by-phase
    rounds: [1]
    status: resolved
  - key: d5-discovery-premise-false
    rounds: [1]
    status: resolved
  - key: d5-discover-signal-unbounded
    rounds: [1]
    status: resolved
  - key: d7-unmatched-turn-start-no-release
    rounds: [1]
    status: resolved
  - key: d7-state-lists-not-exhaustive
    rounds: [1]
    status: resolved
  - key: d5-consume-wakes-loop-twice
    rounds: [1]
    status: resolved
  - key: note-contradicts-d7-idleprobe
    rounds: [1]
    status: resolved
  - key: fsnotify-pin-only-in-assumption
    rounds: [1]
    status: resolved
  - key: open-risks-restate-decisions
    rounds: [1]
    status: resolved
  - key: d11-hook-cwd-drifts
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d11-pointer-names-stale-role-file
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d11-hook-lyx-binary-unstated
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: d4-resize-wake-unbounded
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised ten findings, three BLOCKING, clustered on D11's skip rule and prompt choice, D5's discovery premise and bound, and D7's turn-start release.
Round 2 found those settled and raised four new findings, none on a round-1 key: the D11 hook's cwd (`d11-hook-cwd-drifts`), the role file it names before the reload re-renders it (`d11-pointer-names-stale-role-file`), the hook's unstated `lyx` binary (`d11-hook-lyx-binary-unstated`), and D4's resize wake without a Bound (`d4-resize-wake-unbounded`).
The two MEDIUM ones are carried into Open risks.
