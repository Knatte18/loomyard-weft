---
round: 2
ledger:
  - key: segment-color-decisions-uncarried
    rounds: [1]
    status: resolved
  - key: dead-removers-deleted-late
    rounds: [1]
    status: resolved
  - key: open-hedges-in-test-targets
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
  - key: shuttle-stop-entry-lacks-orch-exception
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: color-send-stalls-prompt-on-command-line
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: stale-colorsequence-test-names
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
  - key: newsendcontext-builders-unnamed
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised three keys.
The color decisions are now carried by cards 11 and 12, and the reviewer withdrew the late-deleted-removers point, so both are resolved.
The test-target hedges shrank from three cards to six named instructions in round 2 (F3), so that key stays open at LOW.
Round 2 adds F1 (the `PATTERN-shuttle-stop` entry names only the loom driver exception while orch removes its strand raw), F2 (the start-time color send stalls a launch whose prompt is on the command line), F4 (stale `ColorSequence` test names) and F5 (`newSendContext` named by no earlier card).
