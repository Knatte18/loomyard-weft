---
round: 2
ledger:
  - key: duplicated-logcapture-helper
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
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
    status: open
    class: scope
    severity: LOW
---

Round 1 raised five findings.
Round 2 found four of them fixed: the reconcile failure tests, the `PushRecorded` retry row, the hub-wide unpushed-commit stamp and the `ModeFor` comment.
The log-capture duplication was fixed in `cmd/lyx` and `hubreconcile` but remains in `fabricengine` and `stencilstore` test files the diff touches (F1, BLOCKING, `PATTERN-testkit`).
F2 is new: the two `PushRecorded` conflict rows without a retry do not assert a clean tree or no rebase in progress.
