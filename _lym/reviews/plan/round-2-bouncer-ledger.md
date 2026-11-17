---
round: 2
ledger:
  - key: hook-test-spawns-at-tier0
    rounds: [1]
    status: resolved
  - key: argument-cap-off-by-one
    rounds: [1]
    status: resolved
  - key: card10-misses-subagent-terms-test
    rounds: [1]
    status: resolved
  - key: override-exception-trusts-unmarked-file
    rounds: [1]
    status: resolved
  - key: render-empty-root-branch-unreached
    rounds: [1]
    status: resolved
  - key: above-root-claude-md-blocks-launch
    rounds: [1]
    status: resolved
  - key: config-dir-failure-outcome-missing
    rounds: [1]
    status: resolved
  - key: target-doc-goes-false
    rounds: [1]
    status: resolved
  - key: pattern-background-listed-as-extends
    rounds: [1]
    status: resolved
  - key: hook-run-left-by-killed-prepare
    rounds: [1]
    status: resolved
  - key: hook-env-lacks-claude-project-dir
    rounds: [1]
    status: resolved
  - key: hook-timeout-row-load-bound-no-waitdelay
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: unreadable-file-rows-name-no-input
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings; round 2 raised none of those keys again, so all eleven are resolved.
Three of them (`argument-cap-off-by-one`, `card10-misses-subagent-terms-test`, `hook-run-left-by-killed-prepare`) were examined in round 2 and rejected as defects.
Round 2 raised two new findings.
F1 (BLOCKING design) is card 8's `TestRunPluginHook` timeout row: a measured one-second bound and no stated `WaitDelay`, with the group kill unpinned.
Its fix names a constant, a deadline-by-channel assertion and a marker-file assertion, all local text, so it is mechanical.
F2 (LOW decision) is the unnamed unreadable-file input in cards 7 and 1, fixed by a directory at the path.
