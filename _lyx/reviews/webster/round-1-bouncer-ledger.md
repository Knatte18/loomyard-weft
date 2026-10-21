---
round: 1
ledger:
  - key: replay-tags-after-reset
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unclassified-push-failure-unrecorded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reconcile-verdict-no-slug-filter
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pattern-entries-rewritten-undeclared
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: set-aside-paths-in-dirtiness-check
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resetkeep-failure-reads-dirty
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: retry-failure-hides-cause
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: weft-error-stalls-warp-arm
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 opened eight findings and none is resolved yet.
Three design findings at MEDIUM gate the loop: the set-aside tags created after the weft reset in `replayWeft`, the unclassified push failure that is neither recorded nor notified, and the reconcile verdict that acts on any entry of the run-scope report.
The undeclared rewrite of PATTERN entries is a MEDIUM consistency finding.
Four LOW design findings concern the dirtiness check, the `ResetKeep` error mapping, the hidden retry cause and the weft error that skips the warp arm of `Pull`.
