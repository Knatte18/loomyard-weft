---
round: 4
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
    rounds: [2, 3, 4]
    status: open
    class: consistency
    severity: MEDIUM
  - key: constraints-retired-pattern-ids
    rounds: [2]
    status: resolved
  - key: d6-retag-helper-sharing
    rounds: [3]
    status: resolved
  - key: constraints-integration-testmain-pairs
    rounds: [3, 4]
    status: open
    class: decision
    severity: MEDIUM
---

Round 4 raised two findings, both MEDIUM and neither design class.
`d6-card-verify-tmux-unsettable` remains open: D6 and the acceptance criterion call `tmux` a gate-run tag although D6 Bound says no standing gate runs it (F1).
`constraints-integration-testmain-pairs` remains open and is raised from LOW to MEDIUM: the isolation constraint does not state the tag-set list or that exactly one TestMain compiles per set (F2).
`d6-retag-helper-sharing` is resolved: the review settled it as not a defect.
