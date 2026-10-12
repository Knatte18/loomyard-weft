---
round: 2
ledger:
  - key: shortcut-false-valued-help-word-undocumented
    rounds: [1]
    status: resolved
  - key: overview-test-frames-name-stale-slug
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: cmd-lyx-test-budget-row-behind-test-count
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
---

Round 1 raised one LOW finding on the undocumented false-valued help or json word in `routeWords`; round 2 no longer raises it.
Round 2 raised two BLOCKING consistency findings, both matching the Publish verify failures.
F1: the pause frames in `TestOverviewSource_ReadsThePrimesRunsFromDisk` named the third run `strict` after the merge renamed it `future-key`.
F2: `cmd/lyx` has one more top-level test than its budget row allows.
