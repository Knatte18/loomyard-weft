---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: operator-commit-reroutes-finalize-to-merge-in
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-in-progress-way-forward-unnamed
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: dirty-with-other-aggregated-reasons
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised twelve findings; round 2 did not re-raise any of them, so all are resolved.
Round 2 raised four new findings.
The BLOCKING one is that the way forward names plain git in the parent's code checkout, outside the task worktree, which the refusal spec's way-forward list does not admit.
Two design MEDIUMs concern the route a commit takes at Finalize (reroute to Merge-In, not a direct land) and the unnamed verb for a parked merge.
The LOW covers a dirty parent whose guard error carries further aggregated reasons.
