---
round: 4
ledger:
  - key: d4-already-landed-empty-diff
    rounds: [1]
    status: resolved
  - key: d6-local-weft-vs-archive-tag-precedence
    rounds: [1]
    status: resolved
  - key: d6-bound-contradicts-r3
    rounds: [1]
    status: resolved
  - key: d5-skip-options-and-originless-repo
    rounds: [1]
    status: resolved
  - key: d7-guard-bypass-unbounded
    rounds: [1]
    status: resolved
  - key: d6-fast-forward-rollback
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: resolved
  - key: added-section-mixes-scheduling
    rounds: [1]
    status: resolved
  - key: d4-resume-premise-false
    rounds: [2]
    status: resolved
  - key: d4-unreadable-diff-halts
    rounds: [2]
    status: resolved
  - key: d5-bound-reconcile-checkout
    rounds: [2]
    status: resolved
  - key: d6-skip-options-silent
    rounds: [2]
    status: resolved
  - key: acceptance-gaps-origin-paths
    rounds: [2]
    status: resolved
  - key: d6-ordering-rule-in-notes
    rounds: [2]
    status: resolved
  - key: notes-plan-time-symbols
    rounds: [2]
    status: resolved
  - key: d7-checkout-adopts-archived-weft-unbounded
    rounds: [3]
    status: resolved
  - key: d7-reconcile-raw-warp-bound
    rounds: [3]
    status: resolved
  - key: d6-origin-record-overwrite
    rounds: [3]
    status: resolved
  - key: acceptance-gaps-d7-paths
    rounds: [3]
    status: resolved
  - key: d4-notice-failure-open
    rounds: [3, 4]
    status: open
    class: consistency
    severity: LOW
  - key: d7-checkout-replaces-worktree-config
    rounds: [4]
    status: open
    class: design
    severity: LOW
  - key: d5-d6-repeated-bound
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
  - key: d7-bypass-justification-raw-worktree
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
---

Rounds 1 to 3 gating findings are resolved: round 4 settles the D7 checkout and raw-warp bounds, the D6 origin record and the acceptance clauses for the D7 paths as not defects.
`d4-notice-failure-open` carries over as F1: the Decision now covers the notice, but its no-strand wording is wrong, so it stays open at LOW consistency.
F2 is a LOW design finding asking D7's checkout bound to name that the archived weft replaces the worktree's per-worktree config and run records.
F3 and F4 are NIT wording findings.
