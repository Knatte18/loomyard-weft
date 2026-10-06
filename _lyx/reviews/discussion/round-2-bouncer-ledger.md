---
round: 2
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
    status: open
    class: design
    severity: MEDIUM
  - key: d4-unreadable-diff-halts
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-bound-reconcile-checkout
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-skip-options-silent
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: acceptance-gaps-origin-paths
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d6-ordering-rule-in-notes
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: notes-plan-time-symbols
    rounds: [2]
    status: open
    class: scope
    severity: NIT
---

Round 1's eight findings are resolved: the fresh round 2 review raises none of them again.
The round 1 D4 empty-diff finding is superseded by a deeper one, since landing never merges the weft, so a resumed Finalize recomputes the same non-empty diff and the risk is a duplicated notice, never a lost one.
Round 2 opens seven findings, no BLOCKING.
Three are gating `design` at MEDIUM: the false D4/R2 resume premise, the Stuck halt on an unreadable config diff, and D5's bound that is wrong for reconcile and checkout.
The rest are the D6 skip-option silence, assertion gaps on origin paths, the ordering rule that lives only in the optional Notes, and plan-time symbol lists in the Notes.
