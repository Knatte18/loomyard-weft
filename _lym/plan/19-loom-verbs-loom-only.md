# Card 19 — loom-verbs-loom-only

**Create:**
- `plan:internal/loomcli#refuseUnservedRecipe` -> `func refuseUnservedRecipe(recipe string) error`
- `plan:internal/loomcli#refuseVerbOffRecipe` -> `func refuseVerbOffRecipe(verb, recipe string) error`
- `plan:internal/loomcli#TestArmAt_RefusesOffLoomRecipes` -> `func TestArmAt_RefusesOffLoomRecipes(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#loomCLI.arm`
- `internal/loomcli#loomCLI.armAt`
- `internal/loomcli#reviewTargetDeps`
- `internal/loomcli#defaultReviewTargetDeps`
- `internal/loomcli#resolveReviewTarget`
- `internal/loomcli#TestResolveReviewTarget`
- `internal/loomcli#fakeReviewTarget`
- `internal/loomcli/doc.go`
- `PATTERN.md`
- `docs/overview.md`

**Uses:**
- `internal/loomrecipe#ServedByLoom`
- `internal/loomrecipe#HasEngine`

**Intent:** The `lym loom` verbs serve loom runs only (D2), and a plan verb refuses on a recipe with no plan by its rows (D3).

Verbs that arm a run:
- `loomCLI` gains `loomPath bool`, which `arm`, the `lym loom` subtree's entry, sets before delegating to `armAt`; `ArmAt`, which `lym shed` shares to arm tapestry, darn and pawl runs, leaves it false, so the refusal never sits in `lym shed`'s path.
- With `loomPath` set, `armAt` checks right after `resolveRecipe` and before any wiring, so a refused verb writes nothing: `refuseUnservedRecipe(c.recipe)`, then `refuseVerbOffRecipe(verb, c.recipe)`.
- `refuseUnservedRecipe` refuses a recipe `loomrecipe.ServedByLoom` does not admit. An embedded non-loom recipe, batten, is refused with `loom: run recipe %q is not a loom recipe, and lym loom serves loom runs only; way forward: drive the run with the lym shed verbs, "lym shed status <run-id>", "lym shed step <run-id>" and "lym shed goto <run-id>"`. A recipe naming no embedded recipe, such as a seed recording `beam` (D11), which `lym shed` refuses too, is refused with `loom: run recipe %q names no recipe this lym knows; way forward: remove the run's seed.json and seed the run again with "lym loom start" or "lym shed seed <run-id> --recipe <recipe>"`.
- An unseeded run reads as tapestry, as today, so `lym loom start` is unaffected. Pawl is the one exception (`ServedByLoom`), so batten's Run-Shed, which starts a pawl run with `lym loom start`, and the orch's verbs on beams keep working until `pawl-loom-recipe` removes it.
- `refuseVerbOffRecipe` refuses `validate-plan`, `insert-card`, `remove-card` and `restore-renumber` on a recipe with no `PlanWrite` row: `loom: %s does not apply to this run, whose %q recipe has no Plan-Write row and so no plan; way forward: none is needed, since the run's scope is its decision record, which "lym loom validate-discussion" checks`. Every other verb passes.

Verbs that resolve their own target: `resolveReviewTarget`, which the circling and decision groups share, reads the target's seed recipe through a new `reviewTargetDeps.seedRecipe` seam (`defaultReviewTargetDeps` reads `shedrun.ReadSeed` of the target's self run) and refuses an unserved recipe saying the verb does not apply to that run: `loom: <group> <verb>: run recipe %q is not a loom recipe, so it has no decision record or review perch this verb acts on; way forward: drive the run with the lym shed verbs`, or the re-seed way forward above for a recipe naming no embedded recipe. A nil `seedRecipe` reads the target as tapestry, so a hand-built `reviewTargetDeps` that never sets it keeps today's behaviour; `fakeReviewTarget` gains a recipe per target for the new rows.

Bound: the exception admits pawl alone, and no other non-loom recipe; batten seeds no run `lym loom` addresses. `lym loom checklist` is exempt: it is a read-only status pane the pre-run never arms, it drives and records nothing, and it draws the rows of whatever recipe the binary embeds, so a batten seed shows batten's rows and a `beam` seed the pane's existing unknown-recipe line.

Tests:
- `TestArmAt_RefusesOffLoomRecipes` (tier0, a hand-built location with a seed written by `shedrun.WriteSeed`, or by hand for `beam`): a batten seed is refused naming `lym shed` and a `beam` seed naming the re-seed, each with no status or seed written; `validate-plan`, `insert-card`, `remove-card` and `restore-renumber` on a darn seed are refused saying the verb does not apply; `status` on a pawl seed and `validate-plan` on a tapestry seed pass both checks (the test asserts any error is neither refusal); and `ArmAt` with a batten seed never returns the refusal.
- `TestResolveReviewTarget` gains a row whose target's seed records batten, refused naming `lym shed`, and a pawl row that resolves.

`PATTERN.md` gains the declared entry, in the "Shed and loom" section:

```
- `PATTERN-loom-contract` — Adding a loom recipe: it declares `contract: loom`, checked by Shed's recipe loader; `lym loom` verbs serve loom runs only, bar pawl until it joins; a verb for some loom recipes refuses on others by the recipe's rows, never its name.
```

`docs/overview.md`'s `lym loom` paragraph names the rule.

**ImpactSummary:** Two refusals join the lym loom arming path and the review-target resolver; lym shed's ArmAt path is untouched, and tapestry, darn and pawl runs keep every verb they use.
