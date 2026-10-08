# Card 12 — Coordinator, discussion, plan and landing specs name their segment

**Edit:**
- `internal/loomcli/driverspec.go`
- `internal/loomcli/driverspec_test.go`
- `internal/loomengine/discussion.go`
- `internal/loomengine/discussion_test.go`
- `internal/loomengine/plan.go`
- `internal/loomengine/plan_test.go`
- `internal/loomengine/rework.go`
- `internal/loomengine/rework_test.go`
- `internal/landingshed/describe.go`
- `internal/landingshed/describe_test.go`
- `internal/mergeresolve/spec.go`
- `internal/mergeresolve/mergeresolve_test.go`
- `internal/frictionengine/spec.go`
- `internal/frictionengine/reflect_test.go`

**Uses:**
- `internal/segmentcolor/segmentcolor.go`
- `internal/shuttleengine/spec.go`

**Intent:**
Each spawning module names its role's segment on its launch spec, next to the role and skills it already names (D13, D14), so reed tints the strand and shuttle types its color.
The module owns the choice, as it owns its role names (`PATTERN-agent-name`).

Segments, per D13's table:

- `driverSpec` (loom driver): `segmentcolor.Coordinator`.
- `DiscussionSpec`: `segmentcolor.Discussion`.
- `PlanSpec`: `segmentcolor.Plan`.
- `ReworkSpec`: `segmentcolor.Webster` (rework belongs to the Webster row).
- `DescribeSpec`: `segmentcolor.Describe`.
- `mergeresolve`'s conflict spec and `frictionengine`'s reflection spec: `segmentcolor.Landing`.

Each builder's doc comment gains one line naming the segment and why it is that one, in the style of its existing per-field reasons.

Tests: each package's existing spec test asserts the built spec's `Segment` beside the role it already asserts; no new test function.

**ImpactSummary:** seven spec builders gain one field each; their runs get a color, nothing else changes.
