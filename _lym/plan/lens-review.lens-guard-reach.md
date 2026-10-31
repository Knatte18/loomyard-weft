```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: tracing whether each guard, refusal and test row the 20 cards add is reachable, and whether a guarded site recurs unguarded.

Findings:

1. MEDIUM — Card 6 (D12, tapestry-module-in-loomengine): tapestry's strict load runs on every loom path, so a fault in `tapestry.yaml` alone also refuses darn and pawl runs.
   - Card 6 says "`LoadConfig` fills them from `LoadTapestryConfig` after its own checks". After this card, every `loomengine.LoadConfig` is also a strict `configengine.Load` of `tapestry.yaml`, plus the plan-perch `CheckResolved` check.
   - So a darn run, a pawl run and the lightweight budget reader all refuse on a fault that is only tapestry's. That covers `wire`, `reviewBudget`, `codeBudgetZero` (`internal/loomcli/landingdeps.go` `codeBudgetZero`) and `RecipeOverride`'s successors.
   - Two inputs reach the refusal after every upstream check passes:
     - a tuned `tapestry.plan_hard_max_bounces` below the plan budget;
     - a hub whose `tapestry.yaml` base does not exist yet. Only `start` and `resume` run the reconcile (`internal/loomcli/arm.go` `arm`: `if verb == "start" || verb == "resume"`), so status, step, goto and decision readers in a hub not yet reconciled get loom's "not initialized" refusal (`internal/loomengine/config.go` `LoadConfig`).
   - D12 places the plan-perch check in the `tapestry` module's load and keeps loom's load to the discussion and code perches. The plan cannot both run the check inside loom's load and stay off darn's path.

2. LOW — Card 8 (D2/D8): `New`'s refusal of a recipe without `contract: loom` cannot be reached once card 19 lands.
   - At plan end, `New` is called only from `newShed`, after card 19's `refuseUnservedRecipe` in `armAt` has already refused batten and `beam`.
   - `lym shed` arms batten through `battencli.ArmAt` (`internal/shedcli/table.go` `recipes`), and refuses `beam` as unknown first (`internal/shedcli/cli.go:264` `lookup(seed.Recipe)`). Pawl is branched off before `New`.
   - Card 8 names no test that reaches the refusal, and does not label it a fail-loud wiring guard. It should say it is one.

3. LOW — Card 19 (D2): the `lym loom checklist` verb skips the non-loom refusal.
   - The pre-run skips it (`internal/loomcli/cli.go` `resolvePersistentPreRun`: `cmd.Name() == "checklist"`).
   - It reads the self run's seed and draws `loomrecipe.Rows(seed.Recipe)` for any recipe, batten included (`internal/loomcli/checklist.go:150-159`).
   - Card 19 guards `armAt` and `resolveReviewTarget` only. This verb addresses a run, so it is a recurring site the card leaves unguarded: it neither refuses nor states an exemption for a read-only pane.

4. LOW — Card 18: the new `codeReviewOff` branch (false for a recipe with no `ParentAlign` row) can be reached only by `TestRecipeKeyedWiring`'s direct call.
   - The seam is read only by the Parent-Align row (`internal/loomcli/landingdeps.go` `codeReviewOff` doc), and a recipe without that row never builds one.
   - Tool calls do not reach it at runtime. It carries forward today's dead darn branch rather than removing it.

Tool calls I made:
- Grepped `internal/shedcli` arming and `TestArmFromSeed_RefusalEnvelopeKinds`, to confirm card 2's `beam` row reaches the unknown-recipe refusal; the shed pre-run's lookup order was not in context.
- Grepped `checklistCmd` resolution and the callers of the loomrecipe builders, to check where card 19's refusal and card 8's `New` refusal can be reached; checklist's own resolution and the `New` call sites were not in context.
- Grepped `fsx.AtomicWriteBytes`'s temporary-file naming and the scratch-dir cleanup sites, to confirm card 11's `.tmp-` crash state and card 15's stop marker can occur; the helper's body was not in context.
- Grepped `checklist.go`'s seed and recipe reads and `resume`'s missing-status refusal, to trace checklist's per-recipe path; that file's body was not in context.
