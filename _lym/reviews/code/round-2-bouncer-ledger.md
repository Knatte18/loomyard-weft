---
round: 2
ledger:
  - key: before-build-dirty-skip-counts-against-max
    rounds: [1]
    status: resolved
  - key: head-moved-check-over-linked-files-untested
    rounds: [1]
    status: resolved
  - key: failed-build-map-format-marker-unasserted
    rounds: [1]
    status: resolved
  - key: nested-package-rule-branches-unpinned
    rounds: [1]
    status: resolved
  - key: before-build-dirty-check-not-shown-fresh
    rounds: [1]
    status: resolved
  - key: failure-paths-and-unresolved-verdict-untested
    rounds: [1]
    status: resolved
  - key: stamp-digests-and-gosumhash-unpinned
    rounds: [1]
    status: resolved
  - key: nested-package-rule-departs-from-impactset
    rounds: [1]
    status: resolved
  - key: after-tests-dirty-skip-not-pinned-against-max
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: changedfiles-added-removed-branches-untested
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings, five MEDIUM and three LOW.
Round 2 does not re-raise any of them, so all eight are resolved.
Round 2 raised two LOW coverage gaps: F1 (the after-tests dirty skip's `--max` accounting) and F2 (`changedFiles` added and removed files).
Both fixes are local test additions whose content the findings state.
