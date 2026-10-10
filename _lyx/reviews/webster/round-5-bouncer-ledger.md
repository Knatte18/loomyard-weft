---
round: 5
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
  - key: pattern-md-over-size-cap
    rounds: [4]
    status: resolved
  - key: reconcile-hubwide-no-upstream-stamp-absent
    rounds: [4, 5]
    status: open
    class: design
    severity: LOW
  - key: pullfirst-no-upstream-untested
    rounds: [4]
    status: resolved
  - key: reconcile-pruned-legacy-paths-recorded-file-written
    rounds: [4]
    status: resolved
  - key: stencilcommit-stale-seeding-descriptions
    rounds: [4, 5]
    status: open
    class: consistency
    severity: LOW
  - key: undersubtree-missing-doc-comment
    rounds: [4]
    status: resolved
---

Rounds 1 to 4 closed all keys except the two below.
Round 5 raised no BLOCKING finding; `PATTERN.md` is back under its cap.
Still open, both LOW: the hub-wide board push leaves the stamp absent for a board with no reachable push destination (F1), and the `CommitSeededStencils` doc comment still describes the retired per-run seeding caller (F2).
