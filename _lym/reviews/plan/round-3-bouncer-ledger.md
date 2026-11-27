---
round: 3
ledger:
  - key: off-mode-record-outside-lock
    rounds: [1]
    status: resolved
  - key: prime-window-start-under-next
    rounds: [1]
    status: resolved
  - key: report-update-builder-missing
    rounds: [1]
    status: resolved
  - key: probes-and-recordings-deferred-to-implementation
    rounds: [1]
    status: resolved
  - key: placement-child-registration-and-slug-readers
    rounds: [1, 2]
    status: resolved
  - key: script-runner-calls-unbounded
    rounds: [1]
    status: resolved
  - key: unknown-worktree-row-misses-branch
    rounds: [1]
    status: resolved
  - key: window-command-skipped-on-child-exits
    rounds: [1]
    status: resolved
  - key: tier-table-extends-and-stale-tier-doc
    rounds: [1]
    status: resolved
  - key: mixed-tier-tests-name-no-file
    rounds: [1]
    status: resolved
  - key: query-within-and-locked-places-read-unstated
    rounds: [1]
    status: resolved
  - key: reedcli-window-file-comment-stale
    rounds: [1]
    status: resolved
  - key: overview-package-tree-misses-kwinplace
    rounds: [1]
    status: resolved
  - key: recordopen-keeps-settledby
    rounds: [1]
    status: resolved
  - key: undecodable-places-heal-race
    rounds: [1]
    status: resolved
  - key: vanished-home-branch-untested
    rounds: [1]
    status: resolved
  - key: hidden-verb-bound-incomplete
    rounds: [1]
    status: resolved
  - key: contract-test-covers-part-of-recordings
    rounds: [1]
    status: resolved
  - key: off-mode-report-update-loses-place
    rounds: [2]
    status: resolved
  - key: contract-test-apply-step-skips-by-design
    rounds: [2]
    status: resolved
---

Round 1 opened most of the keys, with its weight in the child flow and planner cards.
Round 2 resolved those except the placement child's registration scope,
and opened two keys for defects the round-1 fixes brought in.

Round 3 resolves the remaining three and raises no finding, so every key is closed.
The deferred KWin probes are settled by the added decision entry for a first probe card.
