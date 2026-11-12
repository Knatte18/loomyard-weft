---
round: 3
ledger:
  - key: d4-push-verdict-unplaced
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-replay-dirty-stop-unbounded
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-push-carry-criteria-missing
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-lock-placement-and-exit-record-accessor
    rounds: [1, 2]
    status: resolved
  - key: d9-task-pass-hub-wide-runproc-registration
    rounds: [1, 2]
    status: resolved
  - key: extra-branch-departure-and-raddle-split
    rounds: [1, 2]
    status: resolved
  - key: d7-unchanged-map-commit-breaks-audit-filter
    rounds: [1, 2]
    status: resolved
  - key: d11-no-map-guard-loses-killed-import
    rounds: [1, 2]
    status: resolved
  - key: d4-code-branch-push-no-reconcile-bound
    rounds: [2]
    status: resolved
  - key: weft-pull-stops-on-uncommitted-maps
    rounds: [2]
    status: resolved
  - key: d7-redundancy-age-drift-misread-unwritten-map
    rounds: [2]
    status: resolved
  - key: d1-d4-writer-path-source-differs
    rounds: [2]
    status: resolved
  - key: d11-bolt-map-commit-named-wrong
    rounds: [2]
    status: resolved
  - key: d6-machine-sharing-premise-env-layer-and-commit
    rounds: [1]
    status: resolved
  - key: d7-reachability-bound-no-evidence
    rounds: [1]
    status: resolved
  - key: d7-skip-condition-narrower-than-planupkeep
    rounds: [1]
    status: resolved
  - key: d8-compact-write-needs-internal-state
    rounds: [1]
    status: resolved
  - key: d11-first-prime-call-unowned-with-upkeep-off
    rounds: [1]
    status: resolved
  - key: d6-refusal-d11-import-unordered
    rounds: [1]
    status: resolved
  - key: d10-layer-wording
    rounds: [1]
    status: resolved
  - key: d7-restates-constraints
    rounds: [1]
    status: resolved
---

Round 3 drops every round 2 key: the review's Rejected section clears the D4 lock, D9, D7 and D11 keys, and no round 3 finding maps to the others.
The two round 2 push and pull keys are closed by their fixes; round 3 raises new gaps in that fix text.
The new gaps are the D4 push verdict (home, exceptions, side effects) and the D5 replay dirty stop after the pre-pull commit.
A third, LOW, finding asks for D4 criteria covering what a task pair's and the prime's push carry.
