```yaml
lens: fix-affected-set
outcome: reported
```

Unchecked findings: F3, F4 and F7. None of them rests on a set of tests, callers or sites, so there was no search to rerun.

**LOW — Card 12 Intent sentence on `template.yaml`'s `step_idle_timeout_min` comment.** F5's premise is "a comment says the loop kills any idle step". Two other comments say the same thing, and no card lists either. Evidence:
- `internal/loomengine/config.go:7`: "step_idle_timeout_min is the minutes a --until-stop step may show no activity before the loop kills it". Card 12 lists `template.yaml` but not `config.go`'s file comment.
- `internal/loomcli/cli.go:80`: the `stepIdleTimeout` field doc says "the window after which a --until-stop step showing no activity is killed". Card 12 extends only `internal/loomcli/doc.go` and edits `specFor` and `agentActivityFor`, not this field.

**LOW — Card 6 Intent sentence "the `Rows` and `RowNames` docs both name `shedrun.RecipeSampler`".** `internal/loomrecipe/doc.go:34` reads "Rows and RowNames serve the tapestry, darn, batten and pawl recipes". That is the package-doc twin of F6's premise, and once card 6 maps sampler it is false. Card 6 lists `loomrecipe/doc.go` under Edit, but its Intent mentions only that file's row path and the New/Routing sentence, so this line is not covered. `doc.go:40` has the same problem: AmendmentTarget is described as serving "the tapestry, darn, batten or pawl recipe".

**LOW — Card 5 Intent: "`RecipeEngines` … its doc naming `recipes.SamplerRecipe` among its sources".** The second line of that doc, `internal/loomrecipe/names.go:16`, says "All three recipes are the loom module's". After the change that count is wrong. The fix names only the source list, so a literal reading leaves "three" in place.

**F1 and F2 covered.** For F1, the only site that enumerates RecipeEngines' sources is `names.go:15` and `:28`, and card 5 now edits it. The other recipe-embedding tests (`bouncerrows_test.go`, `languageskills_test.go`, `gatequiescence_test.go`) iterate fixed per-recipe tables and are not exhaustive guards. For F2, the other budget-only docs are `internal/loomcli/landingdeps.go:390` and `:400`, on `rerouteReviewed` and `codeBudgetZero`. Both functions are already under card 8's Edit list with their behaviour change stated, so they are covered at target level.

Tool calls:
- grep of embedded-recipe enumeration sites, for F1's set: not in context.
- grep of budget-only perch docs, for F2's set: not in context.
- grep of idle-kill statements, for F5's set: not in context.
- grep of loomrecipe recipe-list docs, for F6's set: not in context.
- ls of `contracts/recipes` and grep of per-recipe tests, to check for an exhaustive embed guard (F1): not in context.
