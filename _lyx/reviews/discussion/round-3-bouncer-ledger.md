---
round: 3
ledger:
  - key: way-forward-command-cannot-clear-dirt
    rounds: [1]
    status: resolved
  - key: handover-restep-bypasses-guard
    rounds: [1]
    status: resolved
  - key: routed-from-premise-false
    rounds: [1]
    status: resolved
  - key: merge-record-admit-yields-discard-text
    rounds: [1]
    status: resolved
  - key: refusal-arm-no-spec-class
    rounds: [1]
    status: resolved
  - key: false-reason-for-goto
    rounds: [1]
    status: resolved
  - key: path-list-collapses-to-one-line
    rounds: [1]
    status: resolved
  - key: ac1-contradicts-cap
    rounds: [1]
    status: resolved
  - key: d1-premise-lacks-evidence
    rounds: [1]
    status: resolved
  - key: settled-fact-filed-as-assumption
    rounds: [1]
    status: resolved
  - key: note-points-at-deleted-function
    rounds: [1]
    status: resolved
  - key: deliberation-in-scope
    rounds: [1]
    status: resolved
  - key: way-forward-plain-git-outside-worktree
    rounds: [2]
    status: resolved
  - key: operator-commit-reroutes-finalize-to-merge-in
    rounds: [2]
    status: resolved
  - key: merge-in-progress-way-forward-unnamed
    rounds: [2]
    status: resolved
  - key: dirty-with-other-aggregated-reasons
    rounds: [2]
    status: resolved
  - key: checkout-step-ordered-after-dirt-step
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-in-progress-arm-names-no-checkout
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: merge-in-wording-cited-for-detached-checkout
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: unscoped-pattern-edit
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: no-parked-merge-read-uncovered
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 raised sixteen findings between them; round 3 re-raised none, so all are resolved.
Round 3 raised five new findings.
The gating one is a design MEDIUM: the other-reasons way forward puts the dirt step before the checkout step, so a commit lands on a detached HEAD.
The other four are a consistency MEDIUM on the merge-in-progress arm naming no checkout, two consistency LOWs on a mis-cited Merge-In wording and an unscoped `PATTERN` edit, and a decision LOW on the no-parked-merge read.
