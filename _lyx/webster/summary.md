# Lighter test rules for stencil prose

All three batches recorded `done`.

## Batch 01-replace-stencil-claims (cards 1–3), head `a03bec83c`

The fork deleted `contracts/stencils/claims_test.go` and landed its D1 keeps in `contracts/stencils/govalues_test.go`.
It pruned webster's rendered-prompt test and Burler's prompt tests to Go-filled, Go-chosen and Go-built text.
The digest listed `contracts/stencils/claims_test.go` and `contracts/stencils/govalues_test.go` as deviations, the scope guard's result for files outside the batch's declared targets.
The fork reported one card deviation: the fixer-prompt row in `TestRenderPrompts` asserts `_lyx/plan`, since the prompt renders the plan directory relative to the worktree root.

## Batch 04-treadle-template-tests (cards 4–6), head `3906dc2d3`

The treadle template test now checks each judge template against the verdict words and keys Go parses.
The tackler, driver, Bouncer, loom, alignment and darn prompt tests were pruned, and the driver stencil's shape test was dropped.
The digest listed `internal/treadleengine/template_test.go` as a deviation, where the fork added a small `yamlTagOf` helper.
The fork also reported one card deviation: `TestDarnSpec` has one `filepath.ToSlash(in.SpecsDir)` want instead of two, because the second would duplicate the first.

## Batch 07-directive-tests (cards 7–8), head `4e8c40673`

The directive and orch pointer tests were pruned.
The coverage rule now exempts stencil prose in the plan spec, the plan stencil and `PATTERN-test-economy`, and the review rubrics gained the matching guards.
The plan-review rubric carries the stencil-prose finding as a clause in its existing Test economy bullet.
The digest listed no deviations.

## Verify gate

The plan-level verify has not run yet.
The gate runs it after this turn ends.
