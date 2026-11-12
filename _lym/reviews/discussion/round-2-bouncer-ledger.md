---
round: 2
ledger:
  - key: d4-lock-placement-and-exit-record-accessor
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d9-task-pass-hub-wide-runproc-registration
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: extra-branch-departure-and-raddle-split
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d7-unchanged-map-commit-breaks-audit-filter
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d11-no-map-guard-loses-killed-import
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d4-code-branch-push-no-reconcile-bound
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: weft-pull-stops-on-uncommitted-maps
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-redundancy-age-drift-misread-unwritten-map
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d1-d4-writer-path-source-differs
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d11-bolt-map-commit-named-wrong
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
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

Round 1's thirteen keys: five reappear in round 2, and the other eight are resolved.
The BLOCKING finding is the build-lock placement in D4, a defect the round 1 fix introduced.
D9's registration fix covered passes but not hand calls in a task.
D7 and D11 each keep a local gap.
The arbiter's entries settled the extra-branch question, leaving a mechanical cleanup of two Open-risks lines and D1's rationale.
Round 2 adds five keys: the code-branch push gate, the weft pull on uncommitted maps, redundancy age and drift, the D1/D4 path source, and the bolt map commit.
