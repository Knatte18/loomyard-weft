---
round: 2
ledger:
  - key: d5-package-var-time-seam
    rounds: [1]
    status: resolved
  - key: d6-retag-drops-gates
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: d6-llm-decision-date
    rounds: [1]
    status: resolved
  - key: d2-coverage-blind-to-spawned-binaries
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
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
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: constraints-retired-pattern-ids
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eight findings, and round 2 did not re-raise six of them, so they are marked resolved.
Two reopened in a new form.
`d6-retag-drops-gates` now concerns the compile-only `llm` tier: the Bound and an acceptance bullet promise a gate runs every retagged test, and none runs `llm`.
`d2-coverage-blind-to-spawned-binaries` now concerns the missing rule for detecting a spawning test.
Round 2 added two consistency findings: the card verify cannot run `-tags tmux` from this task, and Constraints name retired PATTERN IDs.
