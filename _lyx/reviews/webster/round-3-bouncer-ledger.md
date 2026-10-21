---
round: 3
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
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
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
    rounds: [2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: settle-unmoved-weft-swallows-read-error
    rounds: [2]
    status: resolved
  - key: refused-restore-untagged
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: final-tip-read-leaves-branch-moved
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 3 keeps two earlier keys open and adds two.
The watchdog idle-window key is still BLOCKING: the round-2 widening to three seconds leaves the publish verify failing, and the review asks for a window derived from the test's own wait budget.
The undeclared PATTERN overview rewrites recur from round 1 as a consistency finding; the fix restores main's text for the undeclared entries.
Three round-2 keys resolve: the set-aside tagging, the `ResetKeep` error mapping and the unmoved-weft read error.
Two keys are new: a refused tip restore leaves the replayed commits without a tag, and a failed final tip read leaves the branch moved.
