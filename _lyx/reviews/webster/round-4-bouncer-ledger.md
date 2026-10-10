---
round: 4
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
    status: open
    class: design
    severity: BLOCKING
  - key: reconcile-hubwide-no-upstream-stamp-absent
    rounds: [4]
    status: open
    class: design
    severity: LOW
  - key: pullfirst-no-upstream-untested
    rounds: [4]
    status: open
    class: scope
    severity: LOW
  - key: reconcile-pruned-legacy-paths-recorded-file-written
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: stencilcommit-stale-seeding-descriptions
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: undersubtree-missing-doc-comment
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
---

Rounds 1 to 3 closed the six earlier keys; round 3 had no findings.
Round 4 follows the merge of `main` into the branch and the Publish verify failure.
It raises one BLOCKING finding, `PATTERN.md` 46 bytes over the cap, plus five LOW or NIT findings on the new `PullThenCommitWritten` and reconcile paths.
