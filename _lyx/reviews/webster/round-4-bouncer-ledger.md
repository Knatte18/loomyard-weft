---
round: 4
ledger:
  - key: replay-tags-after-reset
    rounds: [1, 2]
    status: resolved
  - key: unclassified-push-failure-unrecorded
    rounds: [1]
    status: resolved
  - key: reconcile-verdict-no-slug-filter
    rounds: [1]
    status: resolved
  - key: pattern-entries-rewritten-undeclared
    rounds: [1, 2, 3, 4]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: set-aside-paths-in-dirtiness-check
    rounds: [1]
    status: resolved
  - key: resetkeep-failure-reads-dirty
    rounds: [1, 2]
    status: resolved
  - key: retry-failure-hides-cause
    rounds: [1]
    status: resolved
  - key: weft-error-stalls-warp-arm
    rounds: [1]
    status: resolved
  - key: watchdog-down-up-idle-window
    rounds: [2, 3, 4]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: settle-unmoved-weft-swallows-read-error
    rounds: [2]
    status: resolved
  - key: refused-restore-untagged
    rounds: [3]
    status: resolved
  - key: final-tip-read-leaves-branch-moved
    rounds: [3]
    status: resolved
  - key: gogit-reindex-rebuild-unsynchronized
    rounds: [4]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 4 adds one key and keeps two open.
The new key is the Publish verify failure: `reindexIfPacksChanged` only clears go-git's index, so concurrent shared-lock readers rebuild it unsynchronized and one sees a half-built map.
The review's fix forces the rebuild under the exclusive lock.
The undeclared PATTERN overview rewrites recur for a fourth round, now classed `decision` because restoring main's five lines needs a choice about overview headroom.
The watchdog idle window recurs as a LOW consistency finding asking for a window of twice the test's wait.
The two round-3 keys about the replay tip are not raised again and resolve.
