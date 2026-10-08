---
round: 3
ledger:
  - key: plan-gate-only-bound-not-implementable-on-shared-seams
    rounds: [1]
    status: resolved
  - key: d1-first-card-depends-on-later-checks
    rounds: [1]
    status: resolved
  - key: no-mechanism-turns-exposed-defect-into-card
    rounds: [1]
    status: resolved
  - key: d5-reference-scan-limits-unstated
    rounds: [1]
    status: resolved
  - key: resign-arrow-head-unbounded
    rounds: [1, 2]
    status: resolved
  - key: no-double-report-rule-delete-only
    rounds: [1]
    status: resolved
  - key: goal-overclaims-verdict-identity
    rounds: [1, 2]
    status: resolved
  - key: d2-bound-names-wrong-sequencing-unit
    rounds: [1]
    status: resolved
  - key: restatements-outside-support-log
    rounds: [1, 2]
    status: resolved
  - key: open-risks-promises-unbuilt-rubric-backstop
    rounds: [2]
    status: resolved
  - key: d3-stability-names-wrong-transition
    rounds: [2]
    status: resolved
  - key: d4-d6-both-claim-stencil-edit
    rounds: [2]
    status: resolved
  - key: d5-quarry-name-breaks-pinned-chokepoint
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: scope-omits-second-rubric-finding
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: no-criterion-rename-to-side-handle-redundancy
    rounds: [3]
    status: open
    class: scope
    severity: LOW
---

Round 3 found no repeat of any earlier key, so every earlier entry is marked resolved: round 3's focus entries (Open risks versus D6, D3's handle transition, the stencil sentence, the re-sign arrow's bounds) were checked and settled, and the three round 1 and 2 themes that had persisted did not reappear.
Round 3 raises one new gating finding: D5 places a `quarry.Name` call in `resolvePass`, which the pinned `TestChokepointCallSites_PinnedToTheirGuardedFunctions` forbids; the Edit head's result must also stay out of `CanonicalizeHandles`' canonicalization maps.
Two LOW findings are new: Scope names one Plan-Review rubric finding where D6 carries two, and no acceptance criterion reaches a `Rename` to-side handle in the D3 redundancy check.
