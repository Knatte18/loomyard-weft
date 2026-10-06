---
round: 3
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
    status: open
    class: design
    severity: MEDIUM
  - key: d7-reconcile-raw-warp-bound
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-origin-record-overwrite
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-notice-failure-open
    rounds: [3]
    status: open
    class: decision
    severity: LOW
  - key: acceptance-gaps-d7-paths
    rounds: [3]
    status: open
    class: scope
    severity: LOW
---

Rounds 1 and 2 are closed: round 3's review raises none of their findings again, and its settled focus questions cover the D4 resume premise, the failed diff read, the D6 skip options and the placement of the ordering rules.
Round 2's `d5-bound-reconcile-checkout` is resolved as stated, with D5 and D7 now bounded for `add` and for the unreachable-origin and no-origin-remote cases.
Round 3 opens five findings, no BLOCKING.
Three are gating `design` at MEDIUM and all concern adoption paths with no stated bound: `checkout` adopting an archive-covered origin weft, `reconcile`'s raw-warp path resting on a proof of a live pair that a raw worktree does not give, and D6 overwriting the origin record of a weft adopted from another machine.
The other two are the open failure of the orch notice in D4 and the missing acceptance clauses for the D7 paths.
