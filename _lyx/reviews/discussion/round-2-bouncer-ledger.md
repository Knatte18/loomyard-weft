---
round: 2
ledger:
  - key: publish-skip-seam-evaluation-point
    rounds: [1]
    status: resolved
  - key: tree-scope-config-not-open-map
    rounds: [1]
    status: resolved
  - key: d1-category-premise-false
    rounds: [1]
    status: resolved
  - key: tree-key-normalisation-gaps
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: acceptance-criteria-miss-two-behaviours
    rounds: [1]
    status: resolved
  - key: scope-omits-loom-round-gate-change
    rounds: [1]
    status: resolved
  - key: guards-scan-error-undecided
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d4-rationale-overstates-once
    rounds: [1]
    status: resolved
  - key: editor-path-skips-darn-loader
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-targets-refusal-covers-empty-value
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: remove-last-tree-ordering-unstated
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1's gating config shape, the Publish skip seam's evaluation point, D1's category premise and D4's overstatement are settled, as are the two coverage and scope-wording corrections.

Round 2 reopens the tree-key gaps at MEDIUM: D2's diff read must report raw paths, and D1's refusals must cover an empty segment and the trailing-slash trim.
The same round raises a new MEDIUM, that the hub-wide editor save never runs darn's loader, so D1's rollback premise holds for `--set` only.
The `--guards` scan error stays open at LOW, now widened to the non-content failures.
Two LOW wording findings are new: the empty `tree_scope_verify` must stay accepted, and removing the last tree needs its off ordering stated.
