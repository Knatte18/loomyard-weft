---
round: 3
ledger:
  - key: duplicated-logcapture-helper
    rounds: [1, 2]
    status: resolved
  - key: reconcile-hubconfig-warpbinding-failure-untested
    rounds: [1]
    status: resolved
  - key: pushrecorded-retry-failure-untested
    rounds: [1]
    status: resolved
  - key: hubwide-unpushed-commit-stamp-one-run
    rounds: [1]
    status: resolved
  - key: stale-modefor-comment-smoke-switchseed
    rounds: [1]
    status: resolved
  - key: pushrecorded-no-seed-conflict-clean-tree-unpinned
    rounds: [2]
    status: resolved
---

Round 1 raised five findings, and round 2 found four fixed.
Round 2 left the log-capture duplication open in `fabricengine` and `stencilstore`, and added the unpinned clean-tree assertion on `PushRecorded` conflict rows.
Round 3 found no findings: the touched test files capture logs through the logcapture kit, and every `PushRecorded` conflict row asserts a clean board.
