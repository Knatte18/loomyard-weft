---
round: 2
ledger:
  - key: empty-optional-directives-fail-table-validation
    rounds: [1]
    status: resolved
  - key: empty-scalar-breaks-advisor-count-bound
    rounds: [1]
    status: resolved
  - key: advisor-bounds-stencil-text-only
    rounds: [1]
    status: resolved
  - key: amendment-path-unscoped-built-ahead-of-need
    rounds: [1]
    status: resolved
  - key: advisor-reliance-two-records-no-criterion
    rounds: [1]
    status: resolved
  - key: d3-bound-misses-live-old-binary-writer
    rounds: [1]
    status: resolved
  - key: rejected-alternative-in-d5-rationale
    rounds: [1]
    status: resolved
  - key: opening-stencil-role-keys-disagree
    rounds: [1]
    status: resolved
  - key: discussion-advisors-not-open-map-reconcile-reverts-scalar
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-parent-trust-rule-premise-false-for-orch
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-bound-overstates-checks-advisor-edit-passes
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

All eight round 1 findings were fixed and none was re-raised in round 2.
Round 2 raised three new design findings, all about advisor limits and config handling: reconcile reverting a scalar `discussion_advisors`, D8's false trust-rule premise for the orch parent, and D5's overstated check chain.
The verdict is CONVERGED; the two MEDIUMs are first raised in the closing round and go to `## Open risks`.
