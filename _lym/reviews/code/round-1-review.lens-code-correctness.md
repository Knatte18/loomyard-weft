```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. Each changed function or branch, the case I ran, and whether it is correct. I ran no tests; this is a read-through of the code.

**List**
- `ResolveReview` split fixer fallback, `internal/loomengine/review.go`:
  - Empty `split_fixer_model` with `raddle` set takes `{cfg.Raddle}`, keyed `loom.raddle`: correct.
  - A set `split_fixer_model` is kept: correct.
  - `raddle` empty falls to the shared `fixer_model`: correct.
- `ResolveJudge` split, `internal/loomengine/review.go`:
  - Seed is passed as `""`, so it takes `seed`, then the split judge: correct.
  - An unset `split_judge` takes `judge`: correct.
- `Config.Caps(PerchSplit)` and the `LoadConfig` budget values, `internal/loomengine/caps.go` and `config.go`:
  - `split_max_bounces: null` takes `review_max_bounces`: correct.
  - A split hard cap of 0 takes the run-wide hard cap: correct.
  - A negative value is refused through `reviewbudget` over `reviewPerchNames`: correct.
- `ConfigSpec` and `segmentModelKeys` for the new keys:
  - `split_auto_accept` is a guard through `reviewbudget.Categories`: correct.
  - The fan keys are guards, and the model and judge keys are none: correct.
- `loomPerches` and `loomSegmentPerches` for `Raddle-Bouncer` and `Raddle-Review`: the segment resolves to `PerchSplit`, and the wiring test pins its models, budget and hard cap. Correct.
- `freshRaddleEntry` in an arras history, `internal/loomcli/raddlewiring.go:61`:
  - A Raddle-File Stuck routed to Raddle-Write counts as fresh: correct.
  - A goto into Raddle-Write counts as fresh: correct.
  - The entry from the Motif block, or from the Discussion rows, does not count as fresh: correct.
  - Raddle-Burler-Fix routes to the Bouncer only, never to Raddle-Write, so a fix round is never read as a fresh entry: correct.
- `raddleWrite.firstMode` for arras: `HoldsSplitPerch(arras)` is true, so the mode is `perch` and `raddle_auto_approve` is never read. Correct.
- Cold proposal store on the `raddle` gate in the Raddle-Burler-Fix row: `StoreColdProposal` does nothing once `cold_stored` is set, so a fixed proposal never overwrites it. Correct, but see finding 1.
- `EndRaddleEntry(slug)` for an arras entry, `internal/boardengine/raddlefile.go:265`: it checks no recipe or label and only marks the entry done. Correct.
- `BouncerRunSubdir("Raddle-Bouncer")`, `internal/loomrecipe/bouncerrows.go`: it is found in arras, the third recipe in the list, and returns `split`. Correct.
- `arbiterRecipeDescription` for arras, `internal/loomcli/arbiterprompt.go`:
  - It matches the new branch only when the recipe has both a `RaddleFile` row and a `DiscussionWrite` row, and the Motif block is expanded by then.
  - The plain raddle recipe still gets the old text.
  - The earlier `PlanWrite` branch cannot match arras.
  - Correct.
- `childRecipeSupported` and `unsupportedChildRecipeRefusal`, `internal/battencli/wire.go`: the format verbs match the argument count. The first list has 7 verbs and 7 arguments. The second has 6 type verbs, one `%q` for the project entry's seed, and the final `got` value. Correct.
- `RecipeNames`, the `embeddedRecipes` table, `RecipeEngines`, and the shedcli table entry: all sorted and consistent. Correct.
- The `arras` blocks in the run-profile template:
  - Every ladder step keeps its hard cap at or above its soft cap, so `CheckResolved` passes (speed 0/1, light 1/2, solid 2/3, wide 3/4).
  - A budget of 0 settles the perch approved at once, so the proposal is filed unreviewed. That matches the `template.yaml` comment.
  - Correct.
- `InterruptPolicies` for the three new rows: each is mapped to reinvoke, so the meta test's two-way check holds. Correct.
- `arras-recipe.yaml` routing:
  - Every `on_done` and `on_stuck` edge is routed and nothing dangles.
  - Raddle-File ends the run.
  - The Raddle-Bouncer, Raddle-Burler-Review and Raddle-Burler-Fix rows share `run_subdir: split` and `role_prefix: split`.
  - Correct.

**Findings**

1. LOW, `design`. `contracts/recipes/arras-recipe.yaml:63-111`, with `internal/loomshed/raddlestate.go:67`.
   - Cold-proposal store: the cold copy is taken when the proposal first passes the gate, before the split perch's fixer edits it. It is never refreshed.
   - Raddle-File stages the cold copy beside the final proposal (`raddlefile.go:266`). So a proposal the cold fixer changed reads as changed after the fact. `lym report raddle` says "whether the operator changed the proposal", and in an arras run that would report the split perch's own fixes as operator edits.
   - D4 only requires that a cold proposal is stored. I found no mention in the diff of how the report should treat arras.

2. LOW, `consistency`. `internal/loomengine/review.go`, the `ResolveJudge` call with an empty seed argument.
   - I did not read `resolveSegmentBouncerModels`, so I cannot confirm whether a bad seed value reports a nonexistent `split_seed` key.
   - I am recording it as unverified, not as a defect.

I found no wrong result, swallowed error or inconsistent failure-path state in the changed Go code. The remaining risk is runtime behaviour that only the unrun run-level test would show.

**Tool calls**
1. One `grep` over `internal/boardengine/*.go` for `EndRaddleEntry` and over `internal/loomcli/*.go` for `freshRaddleEntry`, because my context held neither body.
