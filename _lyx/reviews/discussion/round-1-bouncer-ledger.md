---
round: 1
ledger:
  - key: d11-skip-rule-keys-on-hook-event
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d11-prompt-choice-not-fixed-by-phase
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d5-discovery-premise-false
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d5-discover-signal-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-unmatched-turn-start-no-release
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-state-lists-not-exhaustive
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d5-consume-wakes-loop-twice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: note-contradicts-d7-idleprobe
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: fsnotify-pin-only-in-assumption
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: open-risks-restate-decisions
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first review of the discussion pair; ten findings, none carried from an earlier round.
The gating cluster concerns D11 (skip rule and prompt choice), D5 (discovery premise and missing Bound) and D7 (turn-start release).
The rest are consistency and relocation findings in the record.
