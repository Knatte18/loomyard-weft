---
round: 1
focus:
  - "contracts/specs/loom-plan-spec.md and contracts/stencils/loom/ (loom-template-plan.md, loom-rubric-plan-review.md, loom-rubric-darn-review.md), contracts/stencils/darn/darn-template-write.md: the parent edited plan-authoring and plan-review stencils while the task rewrote the spec's deviation union paragraph; does any parent stencil still describe deviations as a UnitPath-mapped file comparison, or contradict the rule that planglyph.ScopeGuard decides what counts as a deviation?"
  - "cmd/lyx/testdata/test-budget.yaml: the parent raised internal/shuttleengines and the task raised internal/planparser; confirm both lines survived the merge and each package's top-level test count still fits its row."
  - "pattern/PATTERN-shuttle-provider-seam.md: the parent added the RecordReaders rule; check whether any task code (planglyph, planparser) reads provider records or engine specifics outside shuttleengines."
  - "internal/planglyph and internal/planparser against the parent's range: does anything the parent changed in these packages or their callers conflict with the new exported refMember accessor or ScopeGuard's file and package coverage?"
---

The two sides touch disjoint packages, so interaction is expected to sit in the shared stencil and spec text and in the test-budget file.
