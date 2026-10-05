# Card 10 — export the slug length limit

**Rename:**
- `internal/boardengine#maxSlugLength` -> `internal/boardengine#MaxSlugLength`

**Edit:**
- `internal/boardengine#validateSlugLength`
- `internal/boardengine/task_test.go`

**Intent:**
Export the slug length limit (D8) so the board CLI's help can format it.
Rename the constant in place in `internal/boardengine/task.go`, retarget `validateSlugLength` to it, and update the subtest names in `task_test.go` that spell the old name.
The value and the refusal message do not change.

**ImpactSummary:** one constant renamed inside boardengine; no behaviour change.

**Commit:** 10: export the slug length limit
