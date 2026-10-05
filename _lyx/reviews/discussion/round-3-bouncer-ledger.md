---
round: 3
ledger:
  - key: d5-package-var-time-seam
    rounds: [1]
    status: resolved
  - key: d6-retag-drops-gates
    rounds: [1, 2]
    status: resolved
  - key: d6-llm-decision-date
    rounds: [1]
    status: resolved
  - key: d2-coverage-blind-to-spawned-binaries
    rounds: [1, 2]
    status: resolved
  - key: d9-socket-removal-race
    rounds: [1]
    status: resolved
  - key: d7-llmkit-exemption-unenforced
    rounds: [1]
    status: resolved
  - key: d8-background-file-count
    rounds: [1]
    status: resolved
  - key: constraints-risks-restatement
    rounds: [1]
    status: resolved
  - key: d6-card-verify-tmux-unsettable
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: constraints-retired-pattern-ids
    rounds: [2]
    status: resolved
  - key: d6-retag-helper-sharing
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: constraints-integration-testmain-pairs
    rounds: [3]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised eight findings and round 2 four; round 3 raised three, none above MEDIUM.
The two recurring design keys from earlier rounds, `d6-retag-drops-gates` and `d2-coverage-blind-to-spawned-binaries`, are not re-raised and are resolved.
`constraints-retired-pattern-ids` was settled in round 3.
Still open: the D6 Bound over-claims that a plan verify runs `-tags tmux` (F1, consistency).
New in round 3: the retag gives no rule for helpers shared across tags (F2, design, gating) and the `!integration` TestMain pairs are not covered by the rekeying (F3, decision).
