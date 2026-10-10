---
round: 3
ledger:
  - key: form-test-payload-token
    rounds: [1]
    status: resolved
  - key: coupling-test-optionality
    rounds: [1]
    status: resolved
  - key: watcher-index-stale-after-deploy
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: runnable-groups-outside-leaf-rules
    rounds: [1]
    status: resolved
  - key: completion-subtree-unannotated-leaf
    rounds: [1]
    status: resolved
  - key: help-replacement-skipped-guards
    rounds: [1, 2]
    status: resolved
  - key: pointer-filter-match-rule
    rounds: [1]
    status: resolved
  - key: json-help-listing-decision
    rounds: [1, 2]
    status: resolved
  - key: record-narrative-tally-double-statement
    rounds: [1]
    status: resolved
  - key: missing-binary-bound-auto-reload
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: group-long-root-module-list
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised nine findings, none BLOCKING.
Round 2 kept three keys and made two BLOCKING.
Round 3 has no BLOCKING finding.

The help-replacement guard gap and the help-listing decision are resolved: round 3's focus review found D1 and its acceptance criteria conforming.
The watcher-binary key persists at MEDIUM: the recorded watcher path matches the orch pane's binary on the `StartRelaunch` branch only (F1).
Two new keys came from the same D4 and D9 text: the missing-binary bound is false for the auto-reload render site, which returns a tick error and does not go idle (F2), and D9's group-`Long` rule contradicts the root's module list that `TestCLITree_EveryCommand` pins (F3).
