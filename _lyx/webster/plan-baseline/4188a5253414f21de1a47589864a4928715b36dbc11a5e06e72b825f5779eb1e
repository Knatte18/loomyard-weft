# Card 11 — darn-builder-and-routing

**Create:**
- `internal/loomrecipe#checkVerifyAttempts`
- `internal/loomrecipe#applyVerifyAttempts`
- `internal/loomrecipe#NewDarn`
- `internal/loomrecipe#DarnRouting`
- `internal/loomrecipe/darnrecipe_test.go`

**Edit:**
- `internal/loomrecipe/doc.go`
- `internal/loomrecipe/interruptpolicy_meta_test.go`
- `internal/loomrecipe/recipe_test.go`
- `internal/loomshed#InterruptPolicies`
- `internal/loomshed/interruptpolicy_test.go`
- `internal/shedrecipe#Env`
- `internal/testkit/envkit#FullEnv`
- `pattern/PATTERN-verified-tree.md`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D1 and D4: the darn shed builder and routing beside loom's.
`Env` gains `DarnVerifyAttempts int`, read by loomrecipe alone, the budget `darn.yaml`'s `verify_attempts` supplies.
`NewDarn` makes the same status-path coherence checks `New` makes, then refuses `env.DarnVerifyAttempts` below 1 through `checkVerifyAttempts` as its first act, with the message `loomrecipe: verify_attempts = <n>; want at least 1; way forward: run "lyx config darn --set verify_attempts=<n>" from the prime, then "lyx loom resume"`, parses `recipes.DarnRecipe`, stamps the budget onto the `Darn` row's `verify` gate entry through `applyVerifyAttempts` (an error when the row or the entry is missing) and builds through `shedbuild.NewShedFrom`;
no review budget applies, since the recipe has no review segment.
`DarnRouting` projects `recipes.DarnRecipe` through `shedbuild.RoutingOf` and reads no budget, so `goto` is never refused by it.
`InterruptPolicies` gains `NameDarn` with `InterruptPolicyReinvoke`, since the single-session adapter re-attaches.
Tests: `darnrecipe_test.go` builds the recipe from `envkit.FullEnv` and pins the row order, engines and producer types against a darn row table, that `PR-Gate`'s on_stuck resolves to `Darn` inside segment `PR-Review`, that the built `Darn` row's `verify` gate carries `DarnVerifyAttempts` as its attempts, and that a budget below 1 is refused with the recorded way forward before any row builds;
`TestRecipe_StructuralCheckHasNoFindings` becomes a table over the two embedded recipes, `recipes.LoomRecipe` and `recipes.DarnRecipe`;
the interrupt-policy meta test unions the rows of `New` and `NewDarn` in both directions, which is the darn rows' interrupt-policy coverage;
`FullEnv` sets `DarnVerifyAttempts: 3`.
The verified-tree background's opening names the told verify command, the plan's in a loom run and `darn.yaml`'s in a darn run, in place of the plan's alone, and its `publish_verify` clause names that told command as `BaseCommand`;
its site list gains the Darn gate.
The refusal spec gains the row `darn.yaml: verify_attempts below 1`, and the loomrecipe doc describes the second builder and its stamped budget.

**ImpactSummary:** One new `Env` field and the closed interrupt-policy table gains one row; loom's builder and routing are untouched.
