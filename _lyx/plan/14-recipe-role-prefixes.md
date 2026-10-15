# Card 14 — recipe-role-prefixes

**Create:**
- `plan:internal/loomrecipe#rolePrefixViolations` -> `func rolePrefixViolations(recipe shedbuild.Recipe) []string`

**Edit:**
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `internal/loomrecipe#TestBouncerRowsCarryOverTheirOwnSegment`
- `internal/loomrecipe#runLoomShuttle`
- `internal/loomrecipe#blockBouncerJudge`
- `internal/loomrecipe#designWrongBouncerJudge`
- `internal/loomrecipe#countRole`
- `internal/loomrecipe#TestEscalation_PerchBudgetOneWithAutoAccept`
- `internal/loomrecipe/doc.go`
- `contracts/stencils#roleOpeningStencils`

**Intent:**
The shipped recipes give every review segment's sessions a segment-prefixed role (D8).
The loom recipe sets `role_prefix: discussion` on the three `Discussion-Review` rows, `plan` on the three `Plan-Review` rows, `webster` on the three `Webster-Review` rows and `merge` on the three `Merge-Review` rows; the darn recipe sets `darn` on the three `Darn-Review` rows.
Each recipe's header comment, beside the shared-segment ring passage, says a review segment's rows share one `role_prefix`, which leads their sessions' roles, so `ly:<slug>:plan-burler-review` names its segment.
A collision of a prefixed role with another role in the worktree is left to reed's `NumberRole`; the color stays on `color_segment`.

`rolePrefixViolations`, a test helper in `bouncerrows_test.go`, returns one message per violation in a parsed recipe: a row of a review segment (a segment holding a Bouncer row) whose engine is Bouncer, BurlerReview or BurlerFix and that sets no `role_prefix`; a segment whose rows set different values; and two segments that set the same value.
`TestBouncerRowsCarryOverTheirOwnSegment`, which already parses both shipped recipes for a per-segment key of their review rows, gains the role-prefix check as subtests, so the package adds no top-level test: it asserts `rolePrefixViolations` finds none in the loom and darn recipes, and that it finds exactly the one expected violation in each of three recipes built from the loom recipe's parse: one with a review row's key removed, one with a fix row's value changed, and one with two segments' values made equal.
Its doc comment says it pins both per-segment keys.

`roleOpeningStencils`'s doc comment says it maps each spawned role to its opening stencils; with the prefix, a review segment's session carries `<prefix>-<key>` for the keys `bouncer-judge`, `bouncer-seed`, `burler-review` and `burler-fix`, so the comment says a key names a role with any segment prefix left off.

The loom recipe test fixtures dispatch on the role, and the judge's role is now prefixed:
`runLoomShuttle`, `blockBouncerJudge` and `designWrongBouncerJudge` recognise a judge by a role ending in `bouncer-judge`, and `countRole` counts the roles ending in its argument, so `TestSequence_FullRunBlocksAtMergeIn`'s count of three judge spawns holds unchanged.
`TestEscalation_PerchBudgetOneWithAutoAccept`'s Webster judge match reads `webster-bouncer-judge`.

`doc.go` names the role-prefix check beside the carry-over one.

**ImpactSummary:** recipe config on every review-segment row of the loom and darn recipes, read only by card 13's decoders; test fakes in `internal/loomrecipe` follow the new role strings, one stencil-map comment in `contracts/stencils` is corrected, and `tools/tokencount` keeps tallying by the recorded role, so its tallies split per segment.

**Commit:** 14: recipes: segment-prefixed review session roles
