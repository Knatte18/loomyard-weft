# Card 2 — rubric-class-mapping

**Edit:**
- `contracts/stencils/loom/loom-rubric-discussion-review.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils/loom/loom-rubric-webster-review.md`
- `contracts/stencils/claims_test.go`

**Intent:** State what `design` means in each loom review segment's own catchment (D4), so a reviewer files the gating class consistently and the judge can read what each class means from the rubric it embeds.

Each rubric gains a `## Finding class` section that names the four classes, points at burler's review step for their generic meanings rather than restating them, and defines `design` for its segment:

- Discussion-Review: is the design right — a wrong or missing decision, a false premise.
- Plan-Review: batching, sequencing, sizing and `verify:` correctness.
- Webster-Review: the implementation matches the plan and the invariants and is correct.

Each section also says a recurring enumeration gap is filed once as a `design` finding about the method, and that class never changes whether a finding is fixed.
The sections add no stencil marker, so `rubric_test.go`'s marker allowlists stay unchanged.

Add one claim per rubric entry in `claims_test.go`: the rubric carries the `## Finding class` heading and names `design`.

**ImpactSummary:** Prompt text only, read by every loom review round and Bouncer pass through the embedded rubric; no Go behaviour changes.

**Commit:** 2: loom rubrics: define the design class per review segment
