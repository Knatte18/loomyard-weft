---
round: 2
ledger:
  - key: d8-precompare-wrong-bytes
    rounds: [1]
    status: resolved
  - key: d8-race-bound-second-window
    rounds: [1]
    status: resolved
  - key: d4-batten-step-seed-site
    rounds: [1]
    status: resolved
  - key: d4-bound-vanished-batten-status
    rounds: [1]
    status: resolved
  - key: d5-malformed-value-unspecified
    rounds: [1]
    status: resolved
  - key: d5-byte-identity-false-premise
    rounds: [1]
    status: resolved
  - key: d6-retire-legacy-envelope-mismatch
    rounds: [1]
    status: resolved
  - key: d2-nonexistent-self-backlink
    rounds: [1]
    status: resolved
  - key: d2-rationale-restates-a1
    rounds: [1]
    status: resolved
  - key: d5-nonint-rule-loader-premise
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-unwire-reader-classification
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: r4-second-window-overstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

All nine round-1 keys are resolved: round 2 raises none of them again, and its focus verdicts confirm the D4, D5, D6 and D8 fixes.
Round 2 raised three new keys.
`d5-nonint-rule-loader-premise` (F1) is the premise the fixer added to D5, which is false for env markers and leaves null undecided.
`d6-unwire-reader-classification` (F2) is bare `fabric unwire`'s reader classification, left undecided.
`r4-second-window-overstated` (F3) is R4 claiming a second window opens where InnerRun may adopt a running child without a spawn.
